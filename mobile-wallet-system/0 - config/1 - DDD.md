### Guia Técnico para Modelagem do Domínio com DDD em um Serviço de Usuários

#### **1. Identificação de Entidades, Agregados e Value Objects**

##### **Entidades**
- **User**
  - **Atributos:**
    - `id` (identificador único)
    - `name` (nome do usuário)
    - `email` (endereço de email - representado por um Value Object)
    - `password` (senha hash)
    - `roles` (associação com a entidade `Role`)
  - **Relações:** 
    - Um usuário pode ter múltiplas permissões através de suas `roles`.

- **Role**
  - **Atributos:**
    - `id` (identificador único)
    - `name` (nome da role, ex.: "Admin", "User")
    - `permissions` (associação com a entidade `Permission`)
  - **Relações:**
    - Cada `Role` pode ter várias permissões.

- **Permission**
  - **Atributos:**
    - `id` (identificador único)
    - `name` (nome da permissão, ex.: "create-user", "delete-user")

##### **Agregados**
- **UserAggregate**
  - **Root:** `User`
  - Contém e gerencia a relação entre o usuário e suas `roles`.

- **RoleAggregate**
  - **Root:** `Role`
  - Gerencia a relação entre `Role` e `Permission`.

##### **Value Objects**
- **EmailAddress**
  - Representa um endereço de email.
  - **Atributos:** `email` (string validada)
  - **Métodos:** `isValid()`, `toString()`.

- **Password**
  - Representa a senha hash.
  - **Atributos:** `hash` (string de hash da senha)
  - **Métodos:** `verify(rawPassword)`.

---

#### **2. Definição de Boundaries Context**

##### **Contextos Delimitados**
- **User Management Context:**
  - Gerencia usuários, autenticação, e autorização.
  - Inclui entidades como `User` e `Role`.

- **Authorization Context:**
  - Focado em roles e permissions.
  - Inclui entidades como `Role` e `Permission`.

##### **Isolamento do Serviço de Usuários**
- Cada contexto pode ser um microsserviço.
- O serviço de usuários não deve acessar diretamente o banco de dados de autorização, mas sim se comunicar via API ou eventos (ex.: publicação de um evento `UserCreated` para sincronizar permissões).

---

#### **3. Implementação de Regras de Negócio**

##### **Exemplos de Regras**
1. Um email deve ser único ao criar ou atualizar um usuário.
   - **Implementação:** Validar na entidade `User` usando o Value Object `EmailAddress`.
2. Um usuário deve ter ao menos uma `Role` associada.
   - **Implementação:** Implementar essa regra dentro do `UserAggregate`.

##### **Local de Implementação**
- **Entidades:** Regras de negócio específicas de uma entidade, como validação de email.
- **Serviços de Domínio:** Regras que abrangem múltiplas entidades, como atribuição de roles.

---

#### **4. Abstração da Persistência**

##### **Configuração de um UserRepository**
- Crie uma interface `UserRepository` no domínio para definir os métodos necessários:
  ```php
  interface UserRepository {
      public function findById(int $id): ?User;
      public function save(User $user): void;
  }
  ```

- A implementação será feita na camada de infraestrutura, utilizando Eloquent:
  ```php
  class EloquentUserRepository implements UserRepository {
      public function findById(int $id): ?User {
          $userModel = UserModel::find($id);
          return $userModel ? $userModel->toDomain() : null;
      }
      public function save(User $user): void {
          $userModel = UserModel::fromDomain($user);
          $userModel->save();
      }
  }
  ```

---

#### **5. Código Exemplo**

##### **Entidade: User**
```php
class User {
    private int $id;
    private string $name;
    private EmailAddress $email;
    private Password $password;
    private array $roles = [];

    public function __construct(int $id, string $name, EmailAddress $email, Password $password) {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
        $this->password = $password;
    }

    public function assignRole(Role $role): void {
        if (!in_array($role, $this->roles)) {
            $this->roles[] = $role;
        }
    }
}
```

##### **Value Object: EmailAddress**
```php
class EmailAddress {
    private string $email;

    public function __construct(string $email) {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email format");
        }
        $this->email = $email;
    }

    public function toString(): string {
        return $this->email;
    }
}
```

##### **Repositório**
```php
class EloquentUserRepository implements UserRepository {
    public function findById(int $id): ?User {
        $userModel = UserModel::find($id);
        return $userModel ? $userModel->toDomain() : null;
    }

    public function save(User $user): void {
        $userModel = UserModel::fromDomain($user);
        $userModel->save();
    }
}
```

---

#### **6. Boas Práticas e Recomendações**
- **Segregação de Contextos:** Mantenha cada contexto delimitado para evitar dependências indesejadas.
- **Regras no Domínio:** Implemente regras de negócio no domínio, não na camada de aplicação.
- **Testes:** Cubra entidades, value objects e serviços com testes unitários.
- **Event-Driven:** Use eventos para comunicação entre microsserviços, reduzindo o acoplamento.
- **Documentação:** Mantenha a documentação do domínio clara e acessível à equipe.

##### **Benefícios**
- **Escalabilidade:** A separação de contextos permite escalar partes específicas do sistema.
- **Manutenção:** Regras claras no domínio tornam o código mais fácil de entender e alterar.
- **Clareza:** A modelagem com DDD e arquitetura hexagonal proporciona um design intuitivo e robusto.