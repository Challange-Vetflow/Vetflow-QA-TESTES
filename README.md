# 🐾 VetFlow

Projeto desenvolvido para o **Challenge FIAP × CLYVO VET** com foco em prevenção veterinária, integração clínica e acompanhamento contínuo da saúde animal.

---

## 📌 Objetivo

O VetFlow busca transformar a jornada veterinária em uma experiência preventiva e integrada, conectando:

- Tutor
- Clínica
- Veterinário
- Plataforma digital

---

## 🚀 Tecnologias Utilizadas

### Backend
- Java 17
- Spring Boot
- .NET 8
- JWT
- Swagger/OpenAPI

### Mobile
- React Native
- Expo

### Banco de Dados
- Oracle Database

### IA & IoT
- Python
- MQTT
- Simulação IoT

### DevOps & Cloud
- Docker
- Docker Compose
- Azure VM Linux

---

## 📱 Funcionalidades

- Cadastro de pets
- Histórico clínico
- Calendário preventivo
- Agendamento de consultas
- Notificações automáticas
- Dashboard para clínicas
- Score de risco experimental

---

## 🏗️ Arquitetura

O projeto foi estruturado utilizando conceitos de:

- TOGAF
- ArchiMate
- Microsserviços
- APIs REST
- Cloud Computing

---

## 📌 Status da Sprint 1

| Módulo | Status |
|---|---|
| API Java Spring Boot | ✅ |
| CRUD Pets/Vacinas | ✅ |
| Swagger/OpenAPI | ✅ |
| Oracle Database | 🔄 |
| API .NET | 📅 |
| React Native | 📅 |
| IA / Visão Computacional | 🧪 |
| IoT MQTT | 🧪 |
| Deploy Azure | 📅 |

---

## 🧪 Testes e QA (Sprint 4)

Esta seção reúne os testes de validação de sistema da API Java do VetFlow, feitos na disciplina de Compliance, Quality Assurance & Tests.

| Parte | Arquivo | Conteúdo |
|---|---|---|
| Teste manual | [`docs/plano-testes-manuais.md`](docs/plano-testes-manuais.md) | Plano de validação de sistema com 19 casos: objetivo, dados de entrada, saída esperada e passos |
| Automação | [`automacao/`](automacao/) | Collection do Postman com 19 requisições automatizadas, e o ambiente para executá-la |

### Aplicação testada

- Aplicação: https://vetflow-java.onrender.com
- Swagger: https://vetflow-java.onrender.com/swagger-ui.html
- Sistema testado: https://github.com/Challange-Vetflow/vetflow-java

### Como executar a automação (Newman)

Pré-requisito: Node.js 18 ou superior.

```bash
npm install -g newman newman-reporter-htmlextra

newman run automacao/VetFlow-API.postman_collection.json \
  -e automacao/VetFlow.postman_environment.json \
  -r cli,htmlextra --reporter-htmlextra-export relatorio/relatorio.html
```

O resumo aparece no terminal e o relatório completo fica em `relatorio/relatorio.html`.

A primeira requisição pode levar até dois minutos, porque a aplicação roda no plano gratuito do Render. Os testes são encadeados (o tutor criado no primeiro caso é usado nos seguintes), então a collection deve ser executada inteira e na ordem.

### Cobertura dos testes

| Grupo | Casos | O que valida |
|---|---|---|
| Autenticação e cadastro | TC01 a TC04 | Cadastro de tutor, e-mail duplicado, login e senha incorreta |
| Controle de acesso e pets | TC05 a TC09 | Restrição por perfil, cadastro de pet, peso inválido, tutor inexistente |
| Clínicas e consultas | TC10 a TC12 | Cadastro de clínica, agendamento e conclusão de consulta |
| Vacinas e medicamentos | TC13 a TC16 | Regra da próxima dose, registro de vacina, prescrição e suspensão de medicamento |
| Relatórios PL/SQL | TC17 e TC18 | Relatório clínico restrito à veterinária |

### Vídeo de demonstração

Configuração, execução e resultados dos testes: https://youtu.be/KrzoUIwg94w

---

## 👨‍💻 Integrantes

| Nome | RM |
|---|---|
| Andrei de Paiva Gibbini | 563061 |
| Pedro Sakai Silva Zambaca | 565956 |
| Pedro Santos Pequini | 561842 |
| Arthur Câmara | 562310 |
| Diogo Cunha | 563654 |

---

## 🔗 Pitch

https://www.youtube.com/watch?v=F3Q-8iLg-78
