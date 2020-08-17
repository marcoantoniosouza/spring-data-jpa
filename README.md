# Spring Data JPA

Segundo uma tradução livre da documentação **Spring Data JPA** torna simples a implementação de repositórios baseados em JPA. Simplificando a maneira que aplicações Spring acessam dados persistentes.

## Mas o que significa JPA?

**JPA** ou **Java Persistence API** consiste na implementação de abstrações de conceitos para implementação de persistência de dados. Essa abstração se dá por meio de mapeamento entre entidades de modelos relacionais para objetos Java.

## Como se dá a implementação desse mapeamento?

### 1. Entidade

Para facilitar o entendimento vamos utilizar um cenário simples de cadastro de produto onde cada produto tem um identificador único, nome, descrição, unidade de medida, estoque e valor unitário.

Sendo assim iremos implementar o Objeto **Produto.java** da seguinte maneira:

```Java
import javax.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name="TB_PRODUTO")
public class Produto {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;
    
    private String nome;
    private String descricao;
    private String und_medida;
    private BigDecimal estoque;
    private BigDecimal valor_unitario;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public String getDescricao() {
        return descricao;
    }

    public void setDescricao(String descricao) {
        this.descricao = descricao;
    }

    public String getUnd_medida() {
        return und_medida;
    }

    public void setUnd_medida(String und_medida) {
        this.und_medida = und_medida;
    }

    public BigDecimal getEstoque() {
        return estoque;
    }

    public void setEstoque(BigDecimal estoque) {
        this.estoque = estoque;
    }

    public BigDecimal getValor_unitario() {
        return valor_unitario;
    }

    public void setValor_unitario(BigDecimal valor_unitario) {
        this.valor_unitario = valor_unitario;
    }
}
```

Logo na definição da classe temos as anotações @Entity e @Table que indicam que a classe define um mapeamento para uma entidade relacional e a tabela para esse mapeamento.

Sendo assim a classe **Produto** está sendo mapeada para a tabela **TB_PRODUTO** no banco de dados.

Prosseguindo na implementação da classe temos a declaração das variáveis, em específico na variável id temos as anotações @Id e @GeneratedValue que indicam que a variável será mapeada para a coluna de chave primária da tabela e que o valor da mesma será gerado automaticamente. Prosseguindo com a implementação dos Getters e Setters para acesso às variáveis.

Dessa forma temos o mapeamento entre a Classe Java e a Entidade do modelo relacional independente de qual banco de dados estamos utilizando. Esse tipo de abstração agiliza o processo de desenvolvimento e remove a obrigatoriedade do desenvolvedor ter proeficiencia completa no dialeto do banco de dados para determinar as tabelas e o acesso ao conteúdo das mesmas.

### 2. Repositórios

Uma vez efetuado o mapeamento da entidade é necessária a criação de um repositório que será responsável pelo acesso ao conteúdo do banco de dados, seja leitura ou escrita.

Esse repositório consistirá em uma classe que extende a interface **JpaRepository** da seguinte maneira:

```Java
import Produto;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    Produto findById(long id);
}
```

Ao extender a interface **JpaRepository**  que recebe a Entidade e o tipo do identificador da mesma já temos vários métodos úteis definidos, sendo  necessários apenas a definição do metodo ***findById*** para termos os métodos necessários para a implementação de um **CRUD**.

### 3. CRUD

O acesso aos dados e por consequência a implementação do **CRUD** (Create, Read, Update e Delete) se dá pela utilização do **Repositório**.

#### 3.1 Create

Para criar um produto através de uma instância do repositório basta chamar o método **save** passando um objeto do tipo produto como parâmetro da seguinte forma:

> produtoRepository.save(produto)

#### 3.2 Read

Podemos tanto listar todos os produtos utilizando o método **findAll** do Repositório da seguinte forma:

> produtoRepository.findAll()

Ou podemos buscar um produto em específico utilizando o método que descrevemos na construção do Repositório **findById** da seguinte forma:

> produtoRepository.findById(id)

#### 3.3 Update

Para fazer uma atualização em um produto basta utilizar o mesmo método utilizado para criar um produto (**save**) passando como parâmetro o objeto Produto que deseja ser atualizado.

> produtoRepository.save(produto)

#### 3.4 Delete

Já para deletar um produto podemos utilizar o método **deleteById** do repositório passando o identificador do produto como parâmetro da seguinte forma:

> produtoRepository.deleteById(id)

### 4. Consultas Específicas

Numa aplicação real as operações do CRUD não são o suficiente e são necessárias, em grande parte do tempo, consultas mais elaboradas e específicas.

A criação dessas consultas podem ser feitas de diversas maneiras, cada uma com seus pontos fortes e fracos. As formas mais comuns são:

#### 4.1 Consultas Declaradas

Uma consulta pode ser declarada conforme o padrão seguido pelo Spring Data JPA sem a necessidade de escrita de nenhum SQL ou qualquer outra linguagem, apenas seguindo os padrões descritos na [documentação](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation) oficial.

Um exemplo já utilizado neste post foi o método descrito no Repoitório **findById** que busca o produto com base no identificador do mesmo.

Caso queira criar uma consulta onde busque por produtos com o valor unitário maior que um valor específico por exemplo podemos criar o método findByValor_unitarioGreaterThan.

#### 4.2 Consultas Nomeadas

O mesmo exemplo de consulta por valor unitário pode ser feito nomeando uma consulta pre-estabelecida na entidade utilizando a anotação **@NamedQuery**  da seguinte forma:

```Java
@Entity
@NamedQuery(name = "Produto.findByValorUnitario",
    query = "select p from Produto p where p.valor_unitario > ?1")
public class Produto{

}
```

Note que a consulta é escrita utilizando um padrão parecido com o SQL mas que na verdade é um padrão específico para JPA chamado JPL.

Dessa forma, após ser nomeada na entidade a consulta pode ser declarada no Repositório como **findByValorUnitario**.

#### 4.3 Usando @Query

Caso não exista a necessidade de nomear a consulta na Entidade pode ser utilizada a anotação **@Query** diretamente no repositório para a escrita do JPL. Dessa forma a mesma constula de busca por valor unitário pode ser feita diretamente no Repositório da seguinte maneira:

```Java
@Query("select p from Produto p where p.valor_unitario > :valor_unitario")
List<Produto> findByValorUnitario(BigDecimal valor_unitario);
```

## Conclusão

Nesse post tivemos a oportunidade de explorar um pouco do Spring Data JPA e como ele pode auxiliar no acesso a dados persistentes dentro de uma aplicação.
