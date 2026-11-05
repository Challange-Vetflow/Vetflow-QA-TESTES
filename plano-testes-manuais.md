# Plano de Testes Manuais — VetFlow (Sprint 4)

Este documento descreve os testes de validação de sistema (nível de sistema, funcional) da API Java do VetFlow.
Como não usamos mais o Azure Boards, o plano fica neste repositório. Cada teste tem objetivo, dados de entrada, dados de saída esperados e passos.

## Ambiente

- **Aplicação:** https://vetflow-java.onrender.com (Swagger em `/swagger-ui.html`)
- **Banco:** Oracle FIAP compartilhado (schema comum com o backend .NET)
- **Usuário VET de teste:** `vet@vetflow.com` / `senha123` (seed)
- **Ferramenta de execução manual:** Swagger UI (botão *Authorize* com Basic) ou Postman (collection em `automacao/`)

## Ordem de execução e dependências

Os testes são encadeados: o resultado de um alimenta o próximo (tutor → pet → clínica → consulta → vacina/medicamento). Execute na ordem.

## Resumo

| ID | Caso de teste | Resultado esperado |
|---|---|---|
| TC01 | Cadastro de tutor (auto-cadastro) | HTTP 201; role = TUTOR; tutorId preenchido. |
| TC02 | E-mail duplicado no cadastro | HTTP 409; mensagem 'Já existe uma conta cadastrada com esse e-mail' (ou equivalente). |
| TC03 | Login do tutor e dados da sessão | HTTP 200; e-mail e role = TUTOR. |
| TC04 | Login com senha incorreta | HTTP 401; mensagem 'E-mail ou senha inválidos.' |
| TC05 | Tutor não acessa a lista de tutores | HTTP 403. |
| TC06 | Veterinária cadastra pet para um tutor | HTTP 201; tutorId e peso conferem. |
| TC07 | Peso inválido bloqueado antes do banco | HTTP 400 (validação antes do banco). |
| TC08 | Pet com tutor inexistente | HTTP 404. |
| TC09 | Tutor cadastra pet para si mesmo | HTTP 201; tutorId = tutor logado. |
| TC10 | Cadastro de clínica | HTTP 201; id da clínica salvo. |
| TC11 | Agendar consulta futura | HTTP 201; completed = false. |
| TC12 | Marcar consulta como realizada | HTTP 200; completed = true. |
| TC13 | Próxima dose antes da aplicação | HTTP 400. |
| TC14 | Registrar vacina válida | HTTP 201. |
| TC14B | Vacina aparece no prontuário do pet | HTTP 200. Lista contém a Antirrábica registrada no TC14. |
| TC15 | Prescrever e suspender medicamento | HTTP 201; status ACTIVE. |
| TC16 | Suspender medicamento em uso | HTTP 200; status SUSPENDED. |
| TC17 | Relatório clínico restrito à veterinária | HTTP 403. |
| TC18 | Relatório clínico pela veterinária | HTTP 200; objeto com indicadores (ex.: petsAtivos). |

## Casos de teste detalhados

### TC01 — Cadastro de tutor (auto-cadastro)

- **Objetivo:** Criar tutor e usuário na mesma transação via procedures.
- **Autenticação:** Sem autenticação
- **Requisição:** `POST /api/auth/register`

**Dados de entrada:** Nome 'Tutor Teste', e-mail único gerado, telefone 11999990000, senha 'senha123'.

```json
{
  "name": "Tutor Teste",
  "email": "{{tutorEmail}}",
  "phone": "11999990000",
  "password": "{{tutorPassword}}"
}
```

**Saída esperada:** HTTP 201; role = TUTOR; tutorId preenchido.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/auth/register` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC02 — E-mail duplicado no cadastro

- **Objetivo:** Garantir que o banco bloqueia e-mail repetido (ORA-20001).
- **Autenticação:** Sem autenticação
- **Requisição:** `POST /api/auth/register`

**Dados de entrada:** E-mail já usado no TC01.

```json
{
  "name": "Tutor Duplicado",
  "email": "{{tutorEmail}}",
  "phone": "11999990001",
  "password": "senha123"
}
```

**Saída esperada:** HTTP 409; mensagem 'Já existe uma conta cadastrada com esse e-mail' (ou equivalente).

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/auth/register` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC03 — Login do tutor e dados da sessão

- **Objetivo:** Validar autenticação do tutor e o perfil retornado.
- **Autenticação:** TUTOR (criado no TC01)
- **Requisição:** `GET /api/auth/me`

**Dados de entrada:** Basic auth com o tutor criado no TC01.

```json
(sem corpo)
```

**Saída esperada:** HTTP 200; e-mail e role = TUTOR.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `GET /api/auth/me` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC04 — Login com senha incorreta

- **Objetivo:** Garantir que credenciais inválidas são recusadas.
- **Autenticação:** Sem autenticação
- **Requisição:** `POST /api/auth/login`

**Dados de entrada:** E-mail do tutor com senha errada.

```json
{
  "email": "{{tutorEmail}}",
  "senha": "senha-errada"
}
```

**Saída esperada:** HTTP 401; mensagem 'E-mail ou senha inválidos.'

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/auth/login` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC05 — Tutor não acessa a lista de tutores

- **Objetivo:** Validar controle de acesso por perfil (apenas VET).
- **Autenticação:** TUTOR (criado no TC01)
- **Requisição:** `GET /api/tutors`

**Dados de entrada:** Tutor autenticado.

```json
(sem corpo)
```

**Saída esperada:** HTTP 403.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `GET /api/tutors` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC06 — Veterinária cadastra pet para um tutor

- **Objetivo:** Cadastro de pet via SP_INSERT_PET com regras de dados válidos.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/pets`

**Dados de entrada:** Nome 'Thor', espécie DOG, nascimento 2020-05-10, peso 28.5 kg, tutorId do TC01.

```json
{
  "name": "Thor",
  "species": "DOG",
  "breed": "Labrador",
  "birthDate": "2020-05-10",
  "weightKg": 28.5,
  "tutorId": "{{tutorId}}"
}
```

**Saída esperada:** HTTP 201; tutorId e peso conferem.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/pets` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC07 — Peso inválido bloqueado antes do banco

- **Objetivo:** Validação de Bean Validation: peso deve ser maior que zero.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/pets`

**Dados de entrada:** weightKg = 0.

```json
{
  "name": "Peso Zero",
  "species": "CAT",
  "birthDate": "2021-01-01",
  "weightKg": 0,
  "tutorId": "{{tutorId}}"
}
```

**Saída esperada:** HTTP 400 (validação antes do banco).

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/pets` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC08 — Pet com tutor inexistente

- **Objetivo:** Garantir que o banco rejeita tutor que não existe (ORA-20003).
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/pets`

**Dados de entrada:** tutorId = UUID zerado (inexistente).

```json
{
  "name": "Fantasma",
  "species": "DOG",
  "birthDate": "2020-01-01",
  "weightKg": 5,
  "tutorId": "00000000-0000-0000-0000-000000000000"
}
```

**Saída esperada:** HTTP 404.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/pets` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC09 — Tutor cadastra pet para si mesmo

- **Objetivo:** Tutor cria pet e o vínculo é forçado ao próprio tutor logado.
- **Autenticação:** TUTOR (criado no TC01)
- **Requisição:** `POST /api/pets`

**Dados de entrada:** Pet sem tutorId, enviado pelo tutor.

```json
{
  "name": "Mel",
  "species": "CAT",
  "breed": "SRD",
  "birthDate": "2022-03-15",
  "weightKg": 4.2
}
```

**Saída esperada:** HTTP 201; tutorId = tutor logado.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/pets` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC10 — Cadastro de clínica

- **Objetivo:** Veterinária cadastra clínica parceira via SP_INSERT_CLINIC.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/clinics`

**Dados de entrada:** Nome, endereço, telefone e e-mail da clínica.

```json
{
  "name": "Clínica Teste Vetflow",
  "address": "Av. Paulista, 1000",
  "phone": "1130000000",
  "email": "clinica.teste@example.com"
}
```

**Saída esperada:** HTTP 201; id da clínica salvo.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/clinics` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC11 — Agendar consulta futura

- **Objetivo:** Agendamento via SP_INSERT_APPOINTMENT (data deve ser futura).
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/appointments`

**Dados de entrada:** Pet e clínica dos casos anteriores; data = hoje + 7 dias às 10:00; tipo CHECKUP.

```json
{
  "petId": "{{petId}}",
  "clinicId": "{{clinicId}}",
  "scheduledAt": "{{dataFutura}}",
  "type": "CHECKUP",
  "notes": "Check-up anual"
}
```

**Saída esperada:** HTTP 201; completed = false.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/appointments` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC12 — Marcar consulta como realizada

- **Objetivo:** Veterinária conclui a consulta agendada.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `PUT /api/appointments/{{appointmentId}}/complete`

**Dados de entrada:** id da consulta do TC11.

```json
(sem corpo)
```

**Saída esperada:** HTTP 200; completed = true.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `PUT /api/appointments/{{appointmentId}}/complete` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC13 — Próxima dose antes da aplicação

- **Objetivo:** Regra de negócio no banco: próxima dose deve ser posterior à aplicação (ORA-20004).
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/vaccines`

**Dados de entrada:** Aplicação 2026-03-10; próxima dose 2026-03-01.

```json
{
  "petId": "{{petId}}",
  "vaccineName": "V10",
  "appliedAt": "2026-03-10",
  "nextDoseAt": "2026-03-01",
  "batch": "L123"
}
```

**Saída esperada:** HTTP 400.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/vaccines` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC14 — Registrar vacina válida

- **Objetivo:** Vacina registrada via SP_INSERT_VACCINE.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/vaccines`

**Dados de entrada:** Vacina antirrábica, aplicada 2026-03-10, próxima dose 2027-03-10.

```json
{
  "petId": "{{petId}}",
  "vaccineName": "Antirrábica",
  "appliedAt": "2026-03-10",
  "nextDoseAt": "2027-03-10",
  "batch": "L456"
}
```

**Saída esperada:** HTTP 201.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/vaccines` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC14B — Vacina aparece no prontuário do pet

- **Objetivo:** Confirma por requisição normal que a vacina registrada aparece no prontuário.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `GET /api/vaccines/by-pet/{{petId}}`

**Dados de entrada:** Id do pet dos casos anteriores.

```json
(sem corpo)
```

**Saída esperada:** HTTP 200. Lista contém a Antirrábica registrada no TC14.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `GET /api/vaccines/by-pet/{{petId}}` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC15 — Prescrever e suspender medicamento

- **Objetivo:** Prescrição via SP_INSERT_MEDICATION e mudança de status.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `POST /api/medications`

**Dados de entrada:** Amoxicilina 250 mg, 12/12h, de 2026-03-10 a 2026-03-20.

```json
{
  "petId": "{{petId}}",
  "name": "Amoxicilina",
  "dosage": "250 mg",
  "frequency": "12/12h",
  "startDate": "2026-03-10",
  "endDate": "2026-03-20"
}
```

**Saída esperada:** HTTP 201; status ACTIVE.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `POST /api/medications` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC16 — Suspender medicamento em uso

- **Objetivo:** Transição de status ACTIVE -> SUSPENDED.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `PUT /api/medications/{{medicationId}}/suspend`

**Dados de entrada:** id do medicamento do TC15.

```json
(sem corpo)
```

**Saída esperada:** HTTP 200; status SUSPENDED.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `PUT /api/medications/{{medicationId}}/suspend` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC17 — Relatório clínico restrito à veterinária

- **Objetivo:** Relatório PL/SQL disponível só para VET; tutor recebe 403.
- **Autenticação:** TUTOR (criado no TC01)
- **Requisição:** `GET /api/reports/resumo-clinico`

**Dados de entrada:** Tutor autenticado.

```json
(sem corpo)
```

**Saída esperada:** HTTP 403.

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `GET /api/reports/resumo-clinico` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.

### TC18 — Relatório clínico pela veterinária

- **Objetivo:** SP_RESUMO_CLINICO retorna dados para a clínica.
- **Autenticação:** VET (vet@vetflow.com)
- **Requisição:** `GET /api/reports/resumo-clinico`

**Dados de entrada:** VET autenticada.

```json
(sem corpo)
```

**Saída esperada:** HTTP 200; objeto com indicadores (ex.: petsAtivos).

**Passos (manual, pelo Swagger ou Postman):**

1. Faça login com a autenticação indicada.
2. Execute `GET /api/reports/resumo-clinico` com o corpo acima (ajuste IDs gerados nos casos anteriores).
3. Confira o status HTTP e os campos da resposta conforme a saída esperada.
