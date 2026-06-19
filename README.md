# Pit Stop — Área do Cliente

Migração do protótipo `pitstop.html` (arquivo único, Babel inline) para um
projeto real: **frontend Vite + React** e **backend Node/Express**, ambos
organizados em camadas seguindo os princípios **SOLID**.

## Como rodar

### Backend
```bash
cd backend
npm install
npm run dev        # http://localhost:3002
```

### Frontend
```bash
cd frontend
npm install
npm run dev         # http://localhost:5173
```

Login de teste: `marcos.oliveira@email.com` / `pitstop123`

---

## Arquitetura — onde está cada princípio SOLID

### Backend (`backend/src`)

```
routes/        → define os endpoints HTTP, chama controllers
controllers/    → traduzem HTTP ↔ service (sem lógica de negócio)
services/       → regras de negócio, recebem repositórios via construtor
repositories/   → acesso a dados (hoje em memória; troque por DB aqui)
domain/         → regras de domínio puras (ex: cálculo de tarifa)
config/         → container.js: liga tudo (Dependency Injection)
middlewares/    → autenticação e tratamento de erros
```

- **S**RP — cada arquivo faz uma coisa: `VehicleService` só cuida de
  veículo, `PaymentService` só de pagamento, etc.
- **O**CP — `domain/FeeCalculator.js` usa **estratégias de tarifa**
  plugáveis. Criar um plano novo = registrar uma estratégia nova, sem
  tocar no motor de cálculo.
- **L**SP — qualquer estratégia de tarifa ou qualquer repositório pode
  ser substituído por outra implementação com a mesma assinatura, sem
  quebrar quem o usa.
- **I**SP — os controllers só recebem do `req` o que precisam (params,
  body), não o objeto inteiro de domínio.
- **D**IP — services dependem de repositórios injetados via construtor
  (veja `config/container.js`), nunca instanciam suas próprias
  dependências. Trocar "array em memória" por PostgreSQL é mudança
  isolada nesse arquivo.

### Frontend (`frontend/src`)

```
api/            → cliente HTTP cru (fetch + token)
services/       → uma função por endpoint, por domínio
hooks/          → estado React conectado aos services
components/     → UI pura (layout, sidebar, login)
pages/          → UI pura por tela, recebe dados via props/hooks
utils/          → formatação (datas, moeda, placa)
```

Mesma lógica do backend, espelhada: os componentes de página não sabem
fazer fetch — eles recebem dados já prontos de hooks, e os hooks não
sabem nada sobre como a UI vai exibir os dados.

## Próximos passos sugeridos
- Trocar os repositórios em memória por Postgres/Prisma (só a pasta
  `repositories/` muda).
- Adicionar testes unitários nos `services/` (já são fáceis de testar
  por receberem dependências via construtor).
- JWT real em vez do token opaco em memória no `AuthService`.
- Deploy no seu Hetzner VPS: backend como serviço Node atrás do Nginx
  (igual ao que você já fez no AutoGest), frontend como build estático
  servido pelo próprio Nginx.
