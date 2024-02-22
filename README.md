# teste-de-unidade
### MSTest
O MSTest é o framework de teste de unidade da Microsoft, projetado para ser usado com código gerenciado, incluindo C#. Ele faz parte do ecossistema do Visual Studio e fornece uma maneira fácil de criar e executar testes unitários.
 
**Características Principais:**
**Atributos de Teste:** O MSTest usa atributos especiais do C# para marcar métodos como testes. Por exemplo, o atributo ```[TestMethod]``` é usado para indicar que um método é um teste.

**Resultados de Teste:** Após a execução dos testes, o Gerenciador de Testes exibe os resultados, indicando quais testes passaram e quais falharam. Ele também fornece detalhes sobre os resultados de cada teste.

## Classe Bank Account 
```using System;

namespace BankAccountNS
{
    /// <summary>
    /// Bank account demo class.
    /// </summary>
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;

        private BankAccount() { }

        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }

        public string CustomerName
        {
            get { return m_customerName; }
        }

        public double Balance
        {
            get { return m_balance; }
        }

        public void Debit(double amount)
        {
            if (amount > m_balance)
            {
                throw new ArgumentOutOfRangeException("amount");
            }

            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }

            m_balance += amount; // intentionally incorrect code
        }

        public void Credit(double amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }

            m_balance += amount;
        }

        public static void Main()
        {
            BankAccount ba = new BankAccount("Mr. Bryan Walton", 11.99);

            ba.Credit(5.77);
            ba.Debit(11.22);
            Console.WriteLine("Current balance is ${0}", ba.Balance);
        }
    }
}

```

**Campos e Propriedades**
Existem dois campos privados na classe:

1. ```m_customerName:``` Armazena o nome do cliente da conta bancária.
2. ```m_balance:``` Armazena o saldo da conta bancária.
Ambos os campos são marcados como readonly, o que significa que seu valor só pode ser atribuído durante a inicialização ou dentro do construtor.

Existem duas propriedades públicas:

```CustomerName:``` Uma propriedade somente leitura que retorna o nome do cliente.
```Balance:``` Uma propriedade somente leitura que retorna o saldo da conta.


**Métodos Debit e Credit:**
1. ```Debit(double amount):``` Debita um determinado valor da conta. Se o valor for maior que o saldo, lança uma exceção.
2. ```Credit(double amount):``` Adiciona um determinado valor ao saldo da conta. Se o valor for negativo, lança uma exceção.

**Método Main**
O método Main é o ponto de entrada do programa. Ele cria uma instância da classe BankAccount, realiza uma transação de crédito e débito, e exibe o saldo atual da conta no console.

## Teste de Unidade: Debit_WithValidAmount_UpdatesBalance

```[TestMethod]
public void Debit_WithValidAmount_UpdatesBalance()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = 4.55;
    double expected = 7.44;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act
    account.Debit(debitAmount);

    // Assert
    double actual = account.Balance;
    Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
}
``` 

**Objetivo do teste**
Este teste tem como objetivo verificar se o método Debit da classe BankAccount está debitando a quantia correta do saldo da conta.

**Atributo ```[TestMethod]```:**

Indica que o método é um método de teste. Este atributo é específico do MSTest e é usado pelo sistema de teste para identificar métodos que devem ser executados como testes.

**Bloco ```Arrange:```**

- Prepara o estado inicial necessário para executar o teste.
- Define beginningBalance como o saldo inicial da conta.
- Define debitAmount como o valor a ser debitado da conta.
- Define expected como o saldo esperado após a operação de débito.
- Cria uma instância da classe BankAccount com o nome do cliente e o saldo inicial.

**Bloco ```Act:```**

- Executa a ação ou operação que está sendo testada.
- Chama o método Debit na instância de BankAccount com o valor debitAmount.

**Bloco ```Assert:```**
- Verifica se o resultado da ação (após o Act) é o esperado.
- ```double actual = account.Balance```: Obtém o saldo atual da conta após a operação de débito.
- ```Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly")```: Compara o saldo atual (actual) com o saldo esperado (expected).
    - O terceiro parâmetro (0.001) é uma tolerância, permitindo uma pequena diferença devido à precisão numérica de valores de ponto flutuante.
    -  O último parâmetro é uma mensagem opcional que será exibida se a asserção falhar.


### Resultado primeiro Teste
O primeiro teste falhou, pois na classe ``` BankAccount ```, existe um erro intencional no método ```Debit ``` onde, em vez de subtrair o valor ```amount``` do saldo ```m_balance``` ele está sendo somado.
![Teste 1](./assets/TesteF.png)

Depois que o código foi alterado no método ```Debit ```, e o valor ```amount``` passou a ser subtraído do saldo ```m_balance```, o teste passou.

![Teste 1](./assets/TesteC.png)

## Refatoração do Código em Teste

Foi adiocionado as seguintes alterações na classe ```BankAccount```:

```
public const string DebitAmountExceedsBalanceMessage = "Debit amount exceeds balance";
public const string DebitAmountLessThanZeroMessage = "Debit amount is less than zero";

``` 

````
if (amount > m_balance)
{
    throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountExceedsBalanceMessage);
}

if (amount < 0)
{
    throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountLessThanZeroMessage);
}
````

Também foi realizada a seguinte alteração no arquivo de teste:
```
using Microsoft.VisualStudio.TestTools.UnitTesting;
using BankAccountNS;

namespace BankAccountNS.Tests
{
    [TestClass]
    public class BankAccountTests
    {
        [TestMethod]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act
            account.Debit(debitAmount);

            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }

        [TestMethod]
        public void Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = -100.00;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act and assert
            Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
        }

        [TestMethod]
        public void Debit_WhenAmountExceedsBalance_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 20.0;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act and assert
            Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
        }

        [TestMethod]
        public void Credit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double creditAmount = 4.55;
            double expected = 16.54;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act
            account.Credit(creditAmount);

            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not credited correctly");
        }

        [TestMethod]
        public void Credit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double creditAmount = -5.00;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act and assert
            Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Credit(creditAmount));
        }
    }
}

```

**Resultados**
Esse Teste foi feito para falhar, pois o método ```Debit```  na classe ```BankAccount``` atualmente não verifica se o valor do débito excede o saldo antes de subtrair o valor do saldo.

Dessa forma, os resultados do teste foi a seguinte: 
![Teste 2](./assets/Teste2.png)

Para que os testes fossem aprovados seria necessário refazer a alteração na classe ```BankAccount```, que foi realizada duranre a etapa de Refatoração do Código de Teste.
