# Padrões de codificação

Estas regras se aplicam ao frontend e ao backend, salvo quando houver uma restrição técnica específica documentada no próprio projeto.

## Não inserir comentários

Não insira comentários no código. O código deve ser escrito de forma clara, com nomes que expressem a intenção e funções pequenas que expliquem o fluxo por si mesmas.

Comentários só são permitidos quando forem absolutamente necessários, por exemplo, para explicar uma expressão regular complexa ou uma decisão técnica que não possa ser expressa no código.

```ts
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function isValidEmail(email: string): boolean {
  return emailPattern.test(email);
}
```

Prefira nomes expressivos e extração de funções:

```ts
function canPublishArticle(article: Article): boolean {
  return article.status === 'draft' && article.authorId !== undefined;
}
```

## Limitar classes e arquivos a 100 linhas

Classes e arquivos `.ts` devem ter, no máximo, 100 linhas. Quando esse limite for atingido, extraia responsabilidades para outras classes, funções ou arquivos coesos.

```ts
class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly paymentService: PaymentService,
  ) {}

  async create(orderData: CreateOrderData): Promise<Order> {
    const order = this.buildOrder(orderData);
    await this.paymentService.authorize(order.total);
    return this.orderRepository.save(order);
  }

  private buildOrder(orderData: CreateOrderData): Order {
    return new Order(orderData.items, orderData.customerId);
  }
}
```

Se a classe ou o arquivo crescer, separe, por exemplo, a validação, a persistência e as regras de negócio em módulos próprios.

## Limitar métodos e funções a 30 linhas

Métodos e funções devem ter, no máximo, 30 linhas. Se o comportamento for maior, divida-o em métodos privados ou funções auxiliares com responsabilidades claras.

```ts
function registerUser(input: RegisterUserInput): User {
  validateUserInput(input);
  const normalizedEmail = normalizeEmail(input.email);
  const passwordHash = hashPassword(input.password);

  return userRepository.create({
    name: input.name.trim(),
    email: normalizedEmail,
    passwordHash,
  });
}
```

Cada função extraída deve continuar tendo uma única responsabilidade e permanecer dentro do limite de 30 linhas.

## Preferir cláusulas de guarda

Não aninhe mais de três níveis de `if`/`else`. Dê preferência a cláusulas de guarda e `early return` para encerrar casos inválidos ou excepcionais antes do fluxo principal.

Evite:

```ts
function processPayment(order?: Order): PaymentResult {
  if (order) {
    if (order.isReady) {
      if (order.total > 0) {
        return charge(order);
      }
    }
  }

  return { success: false };
}
```

Prefira:

```ts
function processPayment(order?: Order): PaymentResult {
  if (!order || !order.isReady || order.total <= 0) {
    return { success: false };
  }

  return charge(order);
}
```

Quando houver vários casos independentes, retorne cedo em cada um e mantenha o caminho de sucesso no menor nível possível de indentação.

## Limitar parâmetros a três

Evite métodos e funções com mais de três parâmetros. Quando vários dados pertencem ao mesmo contexto, agrupe-os em um objeto parâmetro nomeado.

Evite:

```ts
function createUser(
  name: string,
  email: string,
  role: string,
  active: boolean,
): User {
  return userRepository.create({ name, email, role, active });
}
```

Prefira:

```ts
type CreateUserInput = {
  name: string;
  email: string;
  role: string;
  active: boolean;
};

function createUser(input: CreateUserInput): User {
  return userRepository.create(input);
}
```

O objeto parâmetro deve representar um conceito do domínio, e não ser apenas uma forma de esconder parâmetros sem relação entre si.

## Evitar linhas em branco dentro de métodos e funções

Evite linhas em branco dentro de métodos e funções. A organização visual deve ser feita pela extração de funções e por nomes claros. Linhas em branco são permitidas entre membros de uma classe e entre funções de um arquivo.

Evite:

```ts
function calculateTotal(items: Item[]): number {
  const validItems = items.filter((item) => item.isValid);

  const subtotal = validItems.reduce((total, item) => total + item.price, 0);

  return applyDiscount(subtotal);
}
```

Prefira:

```ts
function calculateTotal(items: Item[]): number {
  const validItems = items.filter((item) => item.isValid);
  const subtotal = validItems.reduce((total, item) => total + item.price, 0);
  return applyDiscount(subtotal);
}
```

## Extrair números e strings mágicos

Extraia números e strings usados como regras de negócio, limites, códigos ou chaves para constantes com nomes que esclareçam seus conceitos.

Evite:

```ts
if (user.loginAttempts >= 5) {
  lockUser(user);
}

if (response.status === 404) {
  return null;
}
```

Prefira:

```ts
const MAX_LOGIN_ATTEMPTS = 5;
const NOT_FOUND_STATUS = 404;

if (user.loginAttempts >= MAX_LOGIN_ATTEMPTS) {
  lockUser(user);
}

if (response.status === NOT_FOUND_STATUS) {
  return null;
}
```

Constantes devem ser declaradas em um escopo adequado e ter nomes que expressem o significado do valor, não apenas o seu tipo.

## Declarar variáveis perto do uso

Declare variáveis o mais próximo possível do local onde são utilizadas. Evite declarar valores no início do método quando eles só serão necessários muito depois.

Evite:

```ts
function sendInvoice(order: Order): void {
  const customer = customerRepository.findById(order.customerId);
  const invoice = invoiceService.create(order);
  const recipient = customer.email;
  const message = buildInvoiceMessage(invoice);

  auditService.record(order.id);
  emailService.send(recipient, message);
}
```

Prefira:

```ts
function sendInvoice(order: Order): void {
  const invoice = invoiceService.create(order);
  const message = buildInvoiceMessage(invoice);
  const customer = customerRepository.findById(order.customerId);
  const recipient = customer.email;
  emailService.send(recipient, message);
  auditService.record(order.id);
}
```

Essa proximidade reduz o escopo das variáveis e facilita a leitura do fluxo.

## Manter dados sensíveis fora do código

Nunca coloque dados sensíveis, como chaves de API, tokens, senhas ou credenciais, diretamente no código ou no repositório. Armazene-os em um arquivo `.env` externo, mantenha o `.env` fora do controle de versão e disponibilize um `.env.example` sem valores reais quando necessário.

Evite:

```ts
const paymentApiKey = 'sk_live_123456789';
```

Prefira:

```env
PAYMENT_API_KEY=chave-real-fora-do-repositorio
```

```ts
const paymentApiKey = process.env.PAYMENT_API_KEY;

if (!paymentApiKey) {
  throw new Error('PAYMENT_API_KEY não configurada');
}
```

Não registre valores sensíveis em logs, não os inclua em mensagens de erro e nunca confirme chaves reais em exemplos ou documentação.
