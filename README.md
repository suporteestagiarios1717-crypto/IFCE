# IFCE

# Portal do Aluno — IFCE Campus Umirim

Portal acadêmico de referência com React + Tailwind CSS, API Node.js/Express e PostgreSQL. A interface combina navegação de produto acadêmico com uma direção de arte verde-escura, fotografia de campus, painéis translúcidos com parcimônia e foco visível por teclado.

> **Status da entrega:** código-fonte executável para desenvolvimento e homologação, não uma instalação institucional pronta para produção. Para operação real, o Campus precisa fornecer/validar cadastro acadêmico, contas e perfis de servidores, políticas de privacidade e retenção, termos institucionais, URLs definitivas, regras acadêmicas vigentes e integrações com identidade, Q-Acadêmico/SUAP/SIGAA e infraestrutura Wi-Fi. Não foram incluídos dados reais de estudantes, credenciais institucionais ou integração administrativa simulada.

## Estrutura

```text
portal-ifce-umirim/
├── docker-compose.yml
├── .env.example
├── README.md
├── backend/
│   ├── .env.example
│   ├── package.json
│   ├── db/schema.sql
│   ├── storage/uploads/          # anexos privados, não publicados como estáticos
│   └── src/
│       ├── app.js, server.js
│       ├── config/               # env e pool PostgreSQL
│       ├── controllers/          # validação HTTP e status codes
│       ├── middleware/           # sessão, origem, upload e erros
│       ├── routes/               # endpoints REST
│       ├── services/             # regras de negócio e acesso a dados
│       ├── scripts/seedDemo.js   # conta sintética para desenvolvimento
│       └── utils/                # CPF, hash, prazo e erros
└── frontend/
    ├── .env.example
    ├── package.json
    ├── tailwind.config.js
    ├── vite.config.js
    ├── public/favicon.svg
    ├── public/imagens/campus.jpg  # fotografia local opcional, veja nota abaixo
    └── src/
        ├── App.jsx                # os 10 módulos
        ├── api.js
        ├── components/            # navegação, marca e componentes UI
        └── styles.css
```

## Requisitos

- Node.js 20 ou superior e npm.
- Docker + Docker Compose (ou PostgreSQL 14+ instalado).
- Duas origens locais: frontend `http://localhost:5173`, API `http://localhost:4000`.

## Executar localmente

1. Na raiz, crie o arquivo `.env` a partir de `.env.example`. Altere `POSTGRES_PASSWORD` para um segredo local.
2. Inicie o PostgreSQL:

   ```bash
   docker compose up -d postgres
   ```

   O schema é aplicado automaticamente quando o volume do banco é criado pela primeira vez. Para reaplicar ou atualizar uma base existente, use `npm run db:schema` depois de criar `backend/.env`; o script lê `DATABASE_URL` desse arquivo.

3. Crie `backend/.env` a partir de `backend/.env.example`. Atualize `DATABASE_URL` para usar a mesma senha definida no `.env` da raiz e troque `JWT_SECRET` e `CPF_HASH_SECRET` por valores aleatórios longos. Para o ambiente local, `COOKIE_SECURE=false` é compatível com HTTP localhost.
4. Instale e inicie a API:

   ```bash
   cd backend
   npm install
   npm run db:schema
   npm run seed:demo
   npm run dev
   ```

5. Em outro terminal, configure e inicie o frontend:

   ```bash
   cd frontend
   cp .env.example .env
   npm install
   npm run dev
   ```

6. Abra `http://localhost:5173`.

### Conta sintética para desenvolvimento

`npm run seed:demo` cria/atualiza uma conta de demonstração somente quando `ENABLE_DEMO_SEED=true` e `NODE_ENV` não é `production`:

- E-mail: `aluno.demo@aluno.ifce.edu.br`
- Senha: `UmirimDemo#2026`
- Matrícula demonstrativa: `DEMO-UMI-2026`
- Data de nascimento usada na consulta demonstrativa: `15/02/2004`

O registro e o curso são inteiramente sintéticos, não pertencem a um estudante real e não devem ser copiados para produção. O CPF não fica salvo em texto puro: o schema armazena um HMAC do CPF, com segredo definido em `CPF_HASH_SECRET`.

## Os 10 módulos

1. **Visão geral:** atalho para todos os serviços e resumo do portal.
2. **Matrícula e e-mail:** consulta por CPF + data de nascimento com validação, limite de tentativas, resposta orientativa e auditoria. Em ambiente real, recomenda-se somar CAPTCHA e verificação de posse por e-mail/OTP antes de revelar identificadores.
3. **Requerimentos:** criar, listar, editar e excluir pedidos que ainda estejam pendentes; perfis de servidor podem mudar o status. O protocolo é emitido no banco; com a seção aberta, os estados são consultados automaticamente a cada 20 segundos.
4. **Atendimento:** abrir chamados por categoria, acompanhar status e trocar mensagens com a equipe. Status administrativo exige perfil `STAFF` ou `ADMIN`.
5. **Restaurante:** solicitações por período, verificação de vínculo ativo e endpoint administrativo para analisar/ativar/recusar pedidos.
6. **Sistemas acadêmicos:** links oficiais para Q-Acadêmico e SUAP; o atalho SIGAA só aparece quando `SIGAA_URL` é confirmado e configurado. O portal não captura senhas de sistemas externos.
7. **Solicitações de ensino:** aproveitamento, validação, justificativa de faltas, segunda chamada, colação de grau, formatura e diploma para cursos técnicos ou superiores de tecnologia. Justificativas têm limite de 14 dias e segunda chamada de 30 dias, validados no servidor. Anexos PDF/JPG/PNG até 5 MB têm assinatura de arquivo verificada e ficam fora da pasta estática.
8. **Editais especiais:** links para as páginas institucionais e subscrição interna de interesse em transferidos/diplomados. A subscrição é armazenada no portal; não envia e-mail sem um serviço de notificação configurado e não substitui a inscrição oficial.
9. **Wi-Fi:** registra solicitações de primeiro acesso ou redefinição. O portal **não inventa uma senha que não funcione na rede**. Para emissão real, configure `WIFI_PROVISIONING_URL` e `WIFI_PROVISIONING_TOKEN` para um provisionador confiável via HTTPS. O adaptador espera `POST` com `{ requestId, enrollmentNumber, operation }` e resposta JSON `{ temporaryPassword, reference? }`. A senha só é devolvida uma vez e não é armazenada pelo portal. Sem integração, a solicitação permanece aguardando a TI.
10. **FAQ:** acordeons pesquisáveis a partir de `faq_entries` no PostgreSQL.

## Configurações essenciais

| Variável | Uso |
|---|---|
| `DATABASE_URL` | Conexão PostgreSQL da API. |
| `FRONTEND_ORIGIN` | Origem única autorizada pelo CORS e validação de origem. |
| `JWT_SECRET` | Assinatura da sessão de oito horas, transmitida em cookie `HttpOnly`. |
| `CPF_HASH_SECRET` | HMAC para CPF e hash de IP de auditoria. Preserve o segredo, pois trocar o valor invalida as consultas existentes por CPF. |
| `COOKIE_SECURE` | Deve ser `true` com HTTPS em produção. |
| `QACADEMICO_URL`, `SUAP_URL`, `SIGAA_URL` | URLs externas confirmadas pelo responsável institucional. SIGAA fica oculto quando vazio. |
| `WIFI_PROVISIONING_URL`, `WIFI_PROVISIONING_TOKEN` | Adaptador HTTPS da rede, opcional. |
| `ENABLE_DEMO_SEED` | Somente `true` em desenvolvimento local; a aplicação encerra se estiver ativado em produção. |

## Endpoints principais

- `GET /api/health` — verificação de saúde e conexão do banco.
- `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me` — sessão por cookie protegido.
- `POST /api/students/lookup`, `GET /api/students/me` — consulta protegida de dados acadêmicos.
- `GET|POST /api/requirements`, `PATCH|DELETE /api/requirements/:id` — requerimentos próprios.
- `GET|POST /api/tickets`, `GET|POST /api/tickets/:id/messages` — atendimento.
- `GET|POST /api/restaurant` — acesso ao restaurante.
- `GET|POST /api/education`, `GET /api/attachments/:id` — solicitações de ensino e anexos privados.
- `GET|POST|DELETE /api/editais/subscriptions` — interesses por editais.
- `GET|POST /api/wifi` — solicitações Wi-Fi.
- `GET /api/faq`, `GET /api/config/links` — informações públicas do portal.
- Endpoints de mudança de status estão sob `/api/staff/...` e exigem perfil de servidor.

Respostas de erro usam `{ error: { code, message, details? } }`; IDs são UUID. A API não expõe uploads como diretório público.

## Testes

```bash
cd backend
npm test
```

Os testes unitários cobrem os limites de 14 e 30 dias, evento futuro, data impossível, cálculo da data civil do Ceará e bloqueio de tentativas de login. Antes de uso institucional, adicione testes de integração com PostgreSQL, testes E2E de fluxos completos, revisão de segurança independente, teste de acessibilidade automatizado e manual e testes de carga em ambiente controlado.

## Segurança e implantação institucional

- Não coloque `.env`, credenciais ou dados de alunos no repositório.
- Em produção, use HTTPS, `COOKIE_SECURE=true`, segredos gerados pelo gerenciador de secrets, PostgreSQL com backup e usuário de privilégio mínimo, armazenamento privado e política de retenção dos anexos.
- Configure contas `STAFF` e `ADMIN` por procedimento administrativo seguro; não existe cadastro público de servidor. A tela de estudante depende de contas provisionadas no banco/IdP.
- Configure o proxy confiável (`trust proxy`) conforme a arquitetura real e revise `FRONTEND_ORIGIN`, CORS, CSP, retenção dos logs e política de IP. O rate limit padrão guarda contadores em memória; produção com múltiplas réplicas deve configurar um store compartilhado. As respostas da API usam `Cache-Control: no-store` para reduzir o risco de persistir dados acadêmicos em cache.
- Para matrícula/e-mail, adote prova de posse (OTP), CAPTCHA e controle antifraude conforme política LGPD. O fluxo CPF + data de nascimento neste projeto é uma base de homologação, não uma garantia suficiente para um sistema oficial.
- A emissão de credenciais Wi-Fi depende de um integrador oficial; a consulta ou geração de credencial deve usar infraestrutura de identidade/RADIUS aprovada pelo campus.
- A imagem institucional em `frontend/public/imagens/campus.jpg` não veio entre os arquivos enviados. Coloque ali uma fotografia autorizada do Campus Umirim. Enquanto estiver ausente, a interface tenta usar uma foto da biblioteca publicada no portal oficial do IFCE. Para preparar uma cópia local, execute `./scripts/fetch-campus-image.sh` em um ambiente com acesso à internet e revise a autorização/atribuição antes da publicação. O script não é executado automaticamente.
- Os endereços e contato do campus devem ser verificados antes de qualquer publicação. Referências consultadas: [Portal do Campus Umirim](https://portal.ifce.edu.br/campus/umirim/), [Contatos do Campus Umirim](https://portal.ifce.edu.br/campus/umirim/contatos-do-campus-umirim/), [Sistemas do IFCE](https://portal.ifce.edu.br/sistemas/).

## Licenças e identidade

A marca, o nome e os ativos institucionais pertencem aos respectivos titulares. Este repositório é uma implementação de referência; a implantação oficial exige autorização e revisão do IFCE.
