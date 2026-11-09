# sistemaAlocacao

Sistema web de locadora de automóveis: cadastro de condutores e veículos, reserva de carros e cálculo da cobrança.

> Projeto de faculdade (abril/2022), feito em PHP orientado a objetos seguindo o padrão de nomenclatura numérica proposto pelo professor. Existe uma versão com algumas telas adicionais em [sisitemaAlocacao](https://github.com/lucasmaraia/sisitemaAlocacao) (o nome tem um typo) — os dois repositórios são variações do mesmo trabalho.

## Stack

- PHP orientado a objetos (classes com `mysqli`)
- MySQL
- HTML, CSS e JavaScript

## Organização dos arquivos

O projeto usa uma convenção numérica para as telas de cada módulo:

| Sufixo | Função                                  |
|--------|-----------------------------------------|
| `_100` | Listagem dos registros                  |
| `_200` | Formulário de inclusão                  |
| `_210` | Processa a inclusão                      |
| `_300` | Formulário de alteração (já preenchido) |
| `_310` | Processa a alteração                     |
| `_400` | Exclusão                                 |

Os módulos são `condut_` (condutores), `auto_` (automóveis), `reser_` (reservas) e `cobr_` (cobranças).

Cada módulo tem uma classe correspondente com a lógica de acesso ao banco:

```
Condutores.php → ConectaBD, Incluir, Alterar, Excluir
Automoveis.php
Reservas.php
Cobrancas.php
index.html     → menu principal com os quatro módulos
locauto.sql    → script de criação das tabelas
estilos.css
funcoes.js
```

## Banco de dados

O `locauto.sql` cria:

```sql
create table automoveis (
    renavam int primary key,
    marca varchar(20),
    modelo varchar(20),
    placa varchar(7),
    preco float
);

create table condutores (
    cnh int primary key,
    nome varchar(20),
    nasc date
);

create table reservas (
    numero int primary key,
    saida datetime,
    retorno datetime default null,
    preco float,
    cnh int,
    renavam int
);
```

As reservas referenciam condutores (`cnh`) e automóveis (`renavam`), mas sem chave estrangeira declarada.

## Configuração

A conexão com o banco é lida de variáveis de ambiente, com valores padrão para um MySQL local:

| Variável      | Padrão      |
|---------------|-------------|
| `DB_HOST`     | `localhost` |
| `DB_USER`     | `root`      |
| `DB_PASSWORD` | vazio       |
| `DB_NAME`     | `locauto` |

```bash
export DB_HOST="localhost"
export DB_USER="root"
export DB_PASSWORD="sua-senha"
export DB_NAME="locauto"
```

No Windows (PowerShell), use `$env:DB_USER = "root"`.

## Como rodar

1. Suba um ambiente com PHP e MySQL.
2. Importe o `locauto.sql`.
3. Configure as variáveis de ambiente do banco (veja a seção acima).
4. Sirva a pasta:

```bash
php -S localhost:8000
```

5. Acesse <http://localhost:8000/index.html>.

## Ressalvas (código de 2022)

- ~~**Credenciais no repositório.**~~ **Corrigido:** o método `ConectaBD()` das quatro classes agora lê variáveis de ambiente. As credenciais antigas (um banco ClearDB do Heroku) seguem no histórico de commits, mas o serviço foi descontinuado e elas estão mortas.
- **Conexão repetida em quatro classes.** O `ConectaBD()` é copiado e colado em `Condutores`, `Automoveis`, `Reservas` e `Cobrancas`. O certo seria centralizar em um único arquivo de conexão incluído pelas quatro.
- **SQL injection.** Todas as queries interpolam os atributos do objeto (vindos de `$_REQUEST`) diretamente na string SQL. Nas telas `_300`, o código concatena o parâmetro na cláusula `WHERE` sem nem aspas. O correto são *prepared statements*.
- O `composer.json` está vazio: não há dependências gerenciadas.
- Chaves primárias informadas manualmente (CNH, RENAVAM, número da reserva), sem `AUTO_INCREMENT` onde faria sentido.
