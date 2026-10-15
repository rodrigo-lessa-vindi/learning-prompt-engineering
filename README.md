# Learning Prompt Engineering

🖥️ This repository is dedicated to master prompt engineering, focusing on prompts and agent configuration for assist software developers.

## About the project

Here you will find a place to organize and share common instructions, prompts and AI agents.
Contributions are welcome!

## Project Structure
```ascii
/
.
├── 📜 README.md
├── 📁 .cursor/                  # (Opcional) Regras automáticas específicas da IDE/Harness
│   └── 📁 rules/                # Atalho para regras que a IDE consome nativamente
│       ├── .cursorrules
│       └── 📄 coding-standards.mdc
├── 📁 examples/
│   ├── 📁 prompts/
│   │   └── 📄 dev_companion_log.md
├── 📁 prompts/
│   ├── 📁 roles/
│   │   └── 📄 dev_companion.md
│   ├── 📁 tasks/
│   │   └── 📄 generate_unit_tests.md
│   └── 📄 default_system_instructions.md
├── 📁 docs/
│   ├── 📁 agents/
│   │   ├── 📄 system-prompt.md  # Instruções primárias do agente (.NET context, postura, restrições)
│   │   └── 📄 architecture.md   # Arquitetura padrão do ecossistema .NET (ex: Clean Architecture)
│   ├── 📁 skills/               # Instruções passo a passo para execução de fluxos
│   │   ├── 📄 01-prd.md         # Como analisar/gerar o PRD
│   │   ├── 📄 02-tech-spec.md   # Como criar a especificação técnica em .NET
│   │   └── 📄 03-task-breakdown.md # Como quebrar a especificação em tarefas incrementais
│   ├── 📁 rules/                # Regras e restrições objetivas do projeto
│   │   ├── 📄 coding-dotnet.md  # Convenções C#, ASP.NET Core e boas práticas .NET
│   │   └── 📄 testing-dotnet.md # Frameworks de teste (xUnit, NSubstitute), cobertura e convenções
│   └── 📁 code-standards/       # Princípios gerais de engenharia e Clean Code
│       ├── 📄 clean-code.md     # Princípios de legibilidade, DRY, SOLID, naming
│       └── 📄 error-handling.md # Padrões de exceções e tratamentos em .NET (ProblemDetails, etc)
└── 📄 AGENTS.md                 # Arquivo raiz / Ponto de entrada (Entrypoint) para qualquer Harness

```

## Detalhamento dos Componentes

### 1. Ponto de Entrada Central (AGENTS.md na Raiz)

Muitos harnesses modernos buscam automaticamente por um arquivo de índice na raiz para carregar contexto. Use este arquivo como o "manual de instruções da documentação":

Define o projeto como um ecossistema .NET (versão alvo, runtime, padrões globais).

Aponta os links/caminhos para os arquivos em docs/ dependendo da tarefa que a IA estiver realizando.

### 2. Pasta docs/skills/ (Fluxos de Trabalho)

Aqui ficam os guias focados em processos sequenciais. Cada arquivo descreve o que o agente deve gerar e em qual formato:

01-prd.md: Template e checklist para definir requisitos funcionais, não-funcionais e critérios de aceite.

02-tech-spec.md: Guia para mapeamento de arquitetura, camadas, entidades do EF Core, endpoints REST/gRPC e dependências.

03-task-breakdown.md: Instruções para gerar um backlog de tarefas pequenas, testáveis e ordenadas.

### 3. Pasta docs/rules/ (Diretrizes do Ecossistema .NET)

Regras diretas e objetivas sobre o código .NET para o agente seguir durante a escrita de código:

coding-dotnet.md: Regras de sintaxe (ex: uso de Primary Constructors, Pattern Matching, record, injeção de dependência via IoC nativo).

testing-dotnet.md: Estrutura dos testes (AAA - Arrange, Act, Assert), nomenclatura de testes (Method_State_ExpectedResult), uso de Mocks e integração.

### 4. Pasta docs/code-standards/ (Princípios de Engenharia)

Conceitos de alto nível independentes de linguagem ou focados na implementação de Clean Code:

clean-code.md: Regras SOLID, tamanho máximo de métodos, imutabilidade por padrão.

error-handling.md: Padrão de respostas de erro (ex: Result<T>, uso de ProblemDetails no ASP.NET).

## How to run the project

### Dicas para Maximizar a Eficiência do Harness

**Modularidade e Referências Cruzadas:** Evite colocar tudo em um único arquivo .md gigante. IAs funcionam melhor com contexto focado. No AGENTS.md, você pode orientar: "Se a tarefa for criar testes, leia primeiramente docs/rules/testing-dotnet.md".

**Uso de Checklists:** Nos arquivos de skills/, use checklists Markdown (- [ ]). IAs são muito boas em seguir etapas quando organizadas em lista de checagem.

**Compatibilidade com Ferramentas (.cursorrules / .cline / .windsurfrules):** Se a sua equipe usa alguma IDE específica, crie um link simbólico ou um arquivo pequeno na raiz apontando diretamente para o AGENTS.md ou importando os arquivos da pasta docs/.

## Let’s Build Together

🚧 If you notice any opportunities for improvement in coding, structure, or best practices, please let me know - I am always eager to learn.

❌ Contributions are welcome! If you’d like to add new prompts, feel free to open a pull request so we can improve this project together.

Show your support
Give a ⭐️ if this repository helped you!
