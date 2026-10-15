# Contexto do projeto

## Padrões de codificação

Consulte [`.agents/rules/code-standards.md`](.agents/rules/code-standards.md) para os padrões de codificação e exemplos aplicáveis ao frontend e ao backend.

Consulte [`.agents/rules/javascript-typescript.md`](.agents/rules/javascript-typescript.md) para as regras de JavaScript e TypeScript, incluindo uso de `const`, comparações estritas, tipagem, arrow functions, ternários e validação com linter.

Para regras específicas de Node.js, assincronismo, event loop, variáveis de ambiente, desligamento, logging, lock files e dependências entre módulos, consulte [`.agents/rules/node.md`](.agents/rules/node.md).

Para regras específicas de componentes, hooks, acessibilidade e estilização React, consulte [`.agents/rules/react.md`](.agents/rules/react.md).

## Regras de testes

Consulte [`.agents/rules/tests.md`](.agents/rules/tests.md) para as regras de testes automatizados, cobertura mínima, princípio FIRST, pirâmide de testes, Vitest, Playwright e organização dos testes E2E.

Este repositório contém dois aplicativos independentes, um frontend e um backend. Os comandos abaixo devem ser executados dentro da pasta do aplicativo correspondente; não existe `package.json` na raiz.

## Estrutura do projeto

Consulte [`.agents/rules/folder-structure.md`](.agents/rules/folder-structure.md) para a organização de pastas e arquivos do frontend, backend e testes.

## Frontend

- Papel: interface web que consome a API do backend e exibe o status da API.
- Tecnologia: React 19, TypeScript, Vite, Tailwind CSS e ESLint.
- Diretório: `frontend/`.
- Desenvolvimento: `http://localhost:5173` (porta padrão do Vite; pode ser alterada pelos argumentos do Vite).
- API consumida atualmente: `http://localhost:3000/health`.

## Backend

- Papel: API HTTP e servidor da aplicação.
- Tecnologia: Node.js, Express 5, TypeScript, CORS e dotenv.
- Diretório: `backend/`.
- Desenvolvimento/produção: `http://localhost:3000` por padrão.
- Porta: definida por `process.env.PORT`; se `PORT` não estiver definida, usa `3000`.
- Health check: `GET /health`.

## Pré-requisitos

É necessário ter Node.js e npm instalados. Cada aplicativo possui seu próprio `package-lock.json`; instale as dependências separadamente em cada diretório.

## Instalação

```bash
cd frontend
npm install

cd ../backend
npm install
```

## Execução em desenvolvimento

Execute frontend e backend em terminais separados:

```bash
# terminal 1
cd backend
npm run dev

# terminal 2
cd frontend
npm run dev
```

O backend ficará em `http://localhost:3000` e o frontend em `http://localhost:5173`. O frontend verifica o backend periodicamente pelo endpoint `/health`.

Para usar outra porta no backend, defina `PORT`, por exemplo:

```bash
cd backend
PORT=3001 npm run dev
```

Nesse caso, também é necessário atualizar a URL usada em `frontend/src/App.tsx`, pois ela está atualmente fixa em `http://localhost:3000/health`.

## Testes, validações e build

### Frontend

Não há framework de testes configurado no frontend. Os comandos disponíveis são:

```bash
cd frontend
npm run lint       # executa o ESLint
npm run typecheck  # verifica os tipos com TypeScript
npm run build      # typecheck + build de produção em dist/
npm run preview    # serve o build de produção localmente
```

### Backend

Estado atual: `npm run build` também falha por causa de `noUnusedParameters` no `backend/src/index.ts` (os parâmetros `req` e `next` não são usados em handlers). Esse problema já existe no código e não foi alterado nesta documentação.

```bash
cd backend
npm run build      # compila TypeScript para dist/
npm test           
npm start          # executa dist/index.js; requer build prévio
```

Para desenvolvimento, use `npm run dev`, descrito acima. O script usa Nodemon e `tsx` para reiniciar o servidor quando arquivos de `src/` mudam.

## Observações para alterações

- Preserve a separação entre `frontend/` e `backend/`; as dependências são instaladas e os scripts são executados por diretório.
- Ao alterar a porta ou o endereço da API, atualize também a URL em `frontend/src/App.tsx` ou extraia essa configuração para uma variável de ambiente.
- Antes de concluir alterações no frontend, execute pelo menos `npm run lint`, `npm run typecheck` e `npm run build` dentro de `frontend/`.
- No backend, execute `npm run build`;
- No frontend, execute `npm run test`;
- No backend, execute `npm run test`;

<critical>SEMPRE SIGA AS REGRAS DE TESTE EM ./agents/rules/tests.md e implemente os testes para o código produzido</critical>