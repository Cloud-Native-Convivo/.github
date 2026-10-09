# Convivo — Plataforma Cloud-Native para Condominios Residenciales

<p align="center">
  <a href="https://cloud-native-convivo.github.io/Frontend-CloudNative/">
    <img src="https://img.shields.io/badge/Demo%20En%20Vivo-Portal%20Residentes-10B981?style=for-the-badge&logo=githubpages&logoColor=white" alt="Demo en Vivo" />
  </a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900?logo=amazon-aws&logoColor=white" alt="AWS Fargate" />
  <img src="https://img.shields.io/badge/IaC-Terraform-844FBA?logo=terraform&logoColor=white" alt="Terraform" />
  <img src="https://img.shields.io/badge/Frontends-React%2019%20%7C%20Angular%2021-61DAFB?logo=react&logoColor=black" alt="Frontends" />
  <img src="https://img.shields.io/badge/BFF-NestJS-E0234E?logo=nestjs&logoColor=white" alt="NestJS BFF" />
  <img src="https://img.shields.io/badge/Backend-FastAPI%20%7C%20Spring%20Boot%204-009688?logo=fastapi&logoColor=white" alt="Backends" />
  <img src="https://img.shields.io/badge/Mensajer%C3%ADa-RabbitMQ-FF6600?logo=rabbitmq&logoColor=white" alt="RabbitMQ" />
</p>

---

## 🏢 Acerca de Convivo

**Convivo** es una plataforma distribuida *cloud-native* diseñada para la administración integral de condominios habitacionales y comunidades residenciales en Chile, alineada con los requisitos de la **Ley N° 21.442 de Copropiedad Inmobiliaria**.

El sistema moderniza la experiencia comunitaria mediante dos canales especializados:
1. **Portal de Residentes**: Autoservicio para reservas de espacios comunes en tiempo real, consulta y pago de gastos comunes, emisión de pases de visita con código QR y libro digital de novedades.
2. **Panel de Administración**: Gestión operativa centralizada para conserjería y comités (cobros, auditorías, bitácoras de incidentes y control de quórum).

---

## 🏛️ Arquitectura General

La solución adopta una arquitectura de microservicios orientada a eventos con desacoplamiento de capas, tolerancia a fallos y doble proveedor de identidad:

```mermaid
flowchart TD
    subgraph Clientes["Canales de Usuario"]
        FR["Portal Residentes\n(React 19 + Vite + Tailwind v4)"]
        FA["Panel Administración\n(Angular 21 + Signals)"]
    end

    subgraph Seguridad["Perímetro & Autenticación"]
        APIGW["Amazon API Gateway\n(HTTP API)"]
        COG["AWS Cognito\n(Residentes + Google OAuth)"]
        ENTRA["Microsoft Entra ID\n(Admins / Conserjería)"]
    end

    subgraph Integracion["Capa de Integración"]
        BFF["BFF — Backend For Frontend\n(NestJS + Circuit Breaker Opossum)"]
    end

    subgraph Dominio["Malla de Microservicios de Dominio (ECS Fargate)"]
        MS_EC["ms-espacios-comunes\n(FastAPI + Python 3.12)"]
        MS_GC["ms-gastos-comunes\n(Spring Boot 4 + Java 25)"]
        MS_CO["ms-condominios\n(FastAPI + Multi-tenancy)"]
        RMQ[("RabbitMQ\n(Event Broker: Sagas / Outbox)")]
    end

    subgraph Gobernanza["Soporte Cloud"]
        CONFIG["config-server-cloud\n(Spring Cloud Config)"]
        DISCOV["discovery-server-cloud\n(Eureka Discovery)"]
    end

    FR -->|JWT Cognito| APIGW
    FA -->|JWT Entra ID| APIGW
    APIGW --> BFF
    BFF --> MS_EC
    BFF --> MS_GC
    BFF --> MS_CO
    MS_EC <-->|Eventos de Reserva| RMQ
    MS_GC <-->|Eventos de Cobro| RMQ
    MS_GC -.->|Configuración Centralizada| CONFIG
    BFF -.->|Resolución de Servicios| DISCOV
```

---

## 📦 Ecosistema de Repositorios

Todos los componentes de Convivo se encuentran modularizados en repositorios independientes con pipelines de CI/CD automatizados:

| Repositorio | Tipo | Stack Principal | Responsabilidad |
| :--- | :--- | :--- | :--- |
| [**`Frontend-CloudNative`**](https://github.com/Cloud-Native-Convivo/Frontend-CloudNative) | Frontend | React 19, Vite, Tailwind v4, TypeScript | Portal web para residentes. Autenticación PKCE con Cognito + Google. [Demo en vivo](https://cloud-native-convivo.github.io/Frontend-CloudNative/). |
| [**`panel-administracion-web`**](https://github.com/Cloud-Native-Convivo/panel-administracion-web) | Frontend | Angular 21, Signals, Tailwind v4 | Interfaz administrativa para conserjería y comités de administración. |
| [**`bff`**](https://github.com/Cloud-Native-Convivo/bff) | Gateway / BFF | NestJS 12, TypeScript, Opossum | Agregación de servicios, validación dual de tokens JWT y Circuit Breaker para resiliencia ante degradaciones. |
| [**`ms-espacios-comunes`**](https://github.com/Cloud-Native-Convivo/ms-espacios-comunes) | Microservicio | FastAPI, Python 3.12, SQLAlchemy | Gestión de quinchos, salas de eventos y piscinas con validación estricta de solapamiento horario (409 Conflict). |
| [**`ms-gastos-comunes`**](https://github.com/Cloud-Native-Convivo/ms-gastos-comunes) | Microservicio | Spring Boot 4, Java 25, RabbitMQ | Facturación, cálculo de prorrateos, conciliación de pagos y patrón Inbox/Outbox transaccional. |
| [**`ms-condominios`**](https://github.com/Cloud-Native-Convivo/ms-condominios) | Microservicio | FastAPI, Python 3.12, SQLAlchemy | Multi-tenancy, administración de comunidades, edificios, torres y unidades residenciales. |
| [**`config-server-cloud`**](https://github.com/Cloud-Native-Convivo/config-server-cloud) | Infraestructura App | Spring Cloud Config, Java 25 | Servidor de configuración distribuida y versionada por perfiles de entorno (`dev`, `prod`). |
| [**`discovery-server-cloud`**](https://github.com/Cloud-Native-Convivo/discovery-server-cloud) | Infraestructura App | Spring Cloud Netflix Eureka, Java 25 | Registro dinámico de instancias y balanceo de carga interno a nivel de aplicación. |
| [**`terraform-cloud`**](https://github.com/Cloud-Native-Convivo/terraform-cloud) | IaC AWS | Terraform (~> 6.0), AWS ECS Fargate, VPC | Aprovisionamiento declarativo de infraestructura en AWS: subnets públicas/privadas, NLB interno, API Gateway, Cognito y Secrets Manager. |

---

## 🛡️ Principios Arquitectónicos y Seguridad

- **Seguridad en Profundidad (OWASP Top 10)**: Tráfico HTTPS/TLS forzado, secretos en AWS Secrets Manager (cero credenciales en texto plano o control de versiones), Lambda Authorizer con validación estandarizada y principio de mínimo privilegio en IAM roles.
- **Autenticación Dual Desacoplada**:
  - **Residentes**: AWS Cognito User Pool con flujo Authorization Code + PKCE y federación Google.
  - **Administración**: Microsoft Entra ID para integración corporativa y control basado en roles (RBAC).
- **Consistencia Eventual & Patrón Saga**: Orquestación asíncrona entre microservicios usando colas durables en RabbitMQ con manejo de compensaciones ante fallas.
- **Observabilidad y Calidad**: Cobertura automatizada en CI (Vitest, pytest, JUnit 5), escaneos de seguridad estática con GitHub CodeQL y formateo estricto con `oxfmt` y linters.

---

## 👥 Equipo de Desarrollo

Proyecto desarrollado para la asignatura **Cloud Native I** en **Duoc UC**:

- **Marcelo Rivera** — [@marceloriv](https://github.com/marceloriv)
- **Karina Pimentel**
