## Spring LDAP

# 1. Introdução
Neste artigo, vamos nos concentrar na integração e configuração do Spring Data LDAP. Para obter uma introdução passo a passo do Spring LDAP, dê uma olhada rápida neste artigo.

Além disso, você pode encontrar a visão geral do guia Spring Data JPA aqui.

# 2. Dependência Maven
Vamos começar adicionando as dependências Maven necessárias:

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-ldap</artifactId>
    <version>1.0.6.RELEASE</version>
</dependency>
```

# 3. Entrada de domínio
O projeto Spring LDAP fornece uma capacidade de mapear entradas LDAP para objetos Java usando Object-Directory Mapping (ODM).

Vamos definir a entidade que será usada para mapear os diretórios LDAP que já foram configurados no artigo Spring LDAP.

```
@Entry(
  base = "ou=users", 
  objectClasses = { "person", "inetOrgPerson", "top" })
public class User {
    @Id
    private Name id;
    
    private @Attribute(name = "cn") String username;
    private @Attribute(name = "sn") String password;

    // standard getters/setters
}
```

@Entry é semelhante a @Entity (de JPA/ORM), que é usado para especificar qual entidade mapeia para a raiz do diretório das entradas LDAP.

Uma classe de entrada deve ter a anotação @Id declarada em um campo do tipo javax.naming.Name que representa o DN da entidade. A anotação @Attribute é usada para mapear campos de classe de objeto para campos de entidade.

# 4. Spring Data Repository
O Spring Data Repository é uma abstração que fornece implementação básica pronta para uso de camadas de acesso a dados para vários armazenamentos de persistência.

O Spring Framework fornece internamente a implementação de operações CRUD para determinada classe no repositório de dados. Podemos encontrar os detalhes completos no artigo Introdução ao Spring Data JPA.

Spring Data LDAP fornece abstração semelhante que fornece a implementação automática de interfaces de repositório que incluem operação CRUD básica para diretórios LDAP.

Além disso, o Spring Data Framework pode criar uma consulta personalizada com base em um nome de método.

Vamos definir nossa interface de repositório que será usada para gerenciar a entrada do usuário:

```
@Repository
public interface UserRepository extends LdapRepository<User> {
    User findByUsername(String username);
    User findByUsernameAndPassword(String username, String password);
    List<User> findByUsernameLikeIgnoreCase(String username);
}
```

Como podemos ver, declaramos uma interface estendendo LdapRepository para a entrada User. Spring Data Framework fornecerá automaticamente a implementação do método CRUD básico, como find (), findAll (), save (), delete (), etc.

Além disso, declaramos alguns métodos personalizados. Spring Data Framework fornecerá a implementação investigando o nome do método com uma estratégia conhecida como Query Builder Mechanism.

# 5. Configuração
Podemos configurar o Spring Data LDAP usando classes @Configuration baseadas em Java ou um namespace XML. Vamos configurar o repositório usando a abordagem baseada em Java:

```
@Configuration
@EnableLdapRepositories(basePackages = "com.baeldung.ldap.**")
public class AppConfig {
}
```

@EnableLdapRepositories sugere que o Spring varra o pacote fornecido em busca de interfaces marcadas como @Repository.

# 6. Lógica de Negócios
Vamos definir nossa classe de serviço que usará o UserRepository para operar em diretórios LDAP:

```
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // business methods
}
```

Agora, vamos explorar uma ação por vez e ver como podemos facilmente realizar essas ações usando Spring Data Repository

6.1. Autenticação de usuário
Agora vamos implementar uma lógica simples para autenticar um usuário existente:


```
public Boolean authenticate(String u, String p) {
    return userRepository.findByUsernameAndPassword(u, p) != null;
}
```

### 6.2 Criação de usuário
A seguir, vamos criar um novo usuário e armazenar um hash de senha:

```
public void create(String username, String password) {
    User newUser = new User(username,digestSHA(password));
    newUser.setId(LdapUtils.emptyLdapName());
    userRepository.save(newUser);
}
```

### 6.3. Modificação do usuário
Podemos modificar um usuário ou entrada existente com o seguinte método:

``` 
public void modify(String u, String p) {
    User user = userRepository.findByUsername(u);
    user.setPassword(p);
    userRepository.save(user);
}
```

# 6.4 Pesquisa de usuário
Podemos pesquisar usuários existentes usando um método personalizado:

```
public List<String> search(String u) {
    List<User> userList = userRepository
      .findByUsernameLikeIgnoreCase(u);
    
    if (userList == null) {
        return Collections.emptyList();
    }

    return userList.stream()
      .map(User::getUsername)
      .collect(Collectors.toList());  
}
```

# 7. Exemplo em ação
Finalmente, podemos testar rapidamente um cenário de autenticação simples:

```
@Test
public void givenLdapClient_whenCorrectCredentials_thenSuccessfulLogin() {
    Boolean isValid = userService.authenticate(USER3, USER3_PWD);
 
    assertEquals(true, isValid);
}
```

# 8. Conclusão
Este rápido tutorial demonstrou os fundamentos da configuração do repositório Spring LDAP e da operação CRUD.

#

Hope that helps. Cheers!
Isac Canedo 😉