
---
## Models
### Criação e Configuração
- **Criação de Model e Migration:** Para criar um model junto com sua migration, utilize o comando:
    ```php
    php artisan make:model NomeDoModel -m
    ```

Onde `-m` indica que uma migration será criada junto ao model. Por convenção, o nome do model é singular e Capitalizado, enquanto a tabela correspondente no banco de dados é plural e em minúsculas.

- **Executando Migrations:** Para aplicar as migrations ao banco de dados, use:
    ```php
    php artisan migrate
    ```

 Isso criará as tabelas no banco de dados conforme definido nas migrations.

### Alterações em Tabelas
- **Adicionando colunas:** Para adicionar novas colunas a uma tabela existente, você pode usar uma migration com o método `Schema::table`, por exemplo:
    ```php
    Schema::table('fornecedores', function (Blueprint $table) {
        $table->string('uf', 2);
        $table->string('email', 150);
    });
    ```

- **Rollback de Migrations:** Se precisar desfazer uma migration, você pode utilizar:
    ```php
    php artisan migrate:rollback
    ```
    Isso executará o método `down` da última migration aplicada.

### Definindo Colunas na Migration
- **Campos nulos e valores padrão:** É possível definir campos que aceitam nulos ou possuem valores padrão diretamente na migration:
    ```php
    Schema::create('produtos', function (Blueprint $table) {
        $table->id();
        $table->string('nome', 100);
        $table->text('descricao')->nullable();
        $table->integer('peso')->nullable();
        $table->float('preco_venda', 8, 2)->default(0.01);
        $table->integer('estoque_minimo')->default(1);
        $table->integer('estoque_maximo')->default(1);
        $table->timestamps();
    });
    ```

### Relacionamentos
- **Chave estrangeira 1:1:** Ao criar um relacionamento um-para-um, defina a chave estrangeira e, opcionalmente, uma restrição de unicidade:
    ```php
    Schema::create('produto_detalhes', function (Blueprint $table) {
        $table->id();
        $table->unsignedBigInteger('produto_id');
        $table->float('comprimento');
        $table->float('largura', 8, 2);
        $table->float('altura', 8, 2);
        $table->timestamps();
        $table->foreign('produto_id')
            ->references('id')
            ->on('produtos');
        $table->unique('produto_id');
    });
    ```

- **Removendo chave estrangeira:** Antes de deletar uma tabela ou coluna que seja chave estrangeira, primeiro remova a restrição:
    ```php
    Schema::table('produtos', function (Blueprint $table) {
        $table->dropForeign(['produtos_unidade_id_foreign']);
        $table->dropColumn('unidade_id');
    });
    ```

- **Relacionamento muitos-para-muitos:** Para relacionamentos N:N, crie uma tabela pivot contendo as chaves estrangeiras das tabelas relacionadas:
    ```php
    Schema::create('produto_filiais', function (Blueprint $table) {
        $table->id();
        $table->unsignedBigInteger('filial_id');
        $table->unsignedBigInteger('produto_id');
        $table->decimal('preco_venda', 8, 2);
        $table->integer('estoque_minimo');
        $table->integer('estoque_maximo');
        $table->timestamps();
        $table->foreign('filial_id')
            ->references('id')
            ->on('filiais');
        $table->foreign('produto_id')
            ->references('id')
            ->on('produtos');
    });
    ```
### Eloquent ORM

#### Métodos de Persistência

### `save()`
Salva um novo registro no banco de dados ou atualiza um existente. É útil para inserções ou atualizações diretas de modelos.

```php
$user = User::find(1);
$user->email = 'novoemail@example.com';
$user->save(); // Atualiza o usuário com ID 1
```

#### Métodos de Recuperação

### `all()`
Retorna todos os registros de uma tabela. Simples e direto para obter uma coleção de todos os modelos.

```php
$users = User::all(); // Retorna todos os usuários
```

### `find($id)`
Busca um registro pelo seu identificador único (ID).

```php
$user = User::find(1); // Busca o usuário com ID 1
```

### `where('column', 'operator', 'value')`
Aplica condições à consulta. É extremamente flexível, permitindo desde comparações simples até as mais complexas.

```php
// Exemplo com operador
$users = User::where('age', '=', 25)->get();

// Exemplo sem especificar o operador (assume igualdade)
$users = User::where('age', 25)->get();

```

### `whereIn('column', array)`
Seleciona registros onde uma coluna tem um valor dentro de um array especificado.

```php
$users = User::whereIn('id', [1, 2, 3])->get(); // Usuários com IDs 1, 2 ou 3
```

### `whereNotIn('column', array)`
O oposto de `whereIn`, seleciona registros que não estão no array especificado.

```php
$users = User::whereNotIn('id', [4, 5])->get(); // Exclui usuários com IDs 4 e 5
```

### `whereBetween('column', [from, to])`
Seleciona registros onde o valor de uma coluna está entre dois valores.

```php
$users = User::whereBetween('age', [20, 30])->get(); // Usuários entre 20 e 30 anos
```

### `whereNotBetween('column', [from, to])`
Seleciona registros onde o valor de uma coluna não está entre dois valores especificados.

```php
$users = User::whereNotBetween('age', [31, 40])->get(); // Exclui usuários entre 31 e 40 anos
```

### `where com duas ou mais condições`
Permite combinar múltiplas condições usando encadeamento de métodos.

```php
$users = User::where('status', '=', 'active')->where('age', '>=', 18)->get();
```

### `orWhere('column', 'operator', 'value')`
Adiciona uma condição "OR" à consulta.

```php
$users = User::where('age', '<', 18)->orWhere('age', '>', 65)->get();
```

### `whereNull('column')`
Seleciona registros onde uma coluna é NULL.

```php
$users = User::whereNull('deleted_at')->get(); // Usuários que não foram excluídos
```

### `whereNotNull('column')`
Seleciona registros onde uma coluna não é NULL.

```php
$users = User::whereNotNull('confirmed_at')->get(); // Usuários que confirmaram o e-mail
```

### `whereDate('column', 'value')`
Compara uma coluna de data com uma data específica.

```php
$users = User::whereDate('created_at', '2020-01-01')->get(); // Usuários criados em 01/01/2020
```

### `whereDay('column', 'value')`, `whereMonth('column', 'value')`, `whereYear('column', 'value')`
Permite comparações específicas por dia, mês ou ano de uma coluna de data.

```php
$users = User::whereMonth('created_at', '12')->get(); // Usuários criados em dezembro de

 qualquer ano
```

### `whereColumn('column1', 'operator', 'column2')`
Compara o valor de duas colunas na mesma tabela.

```php
// Exemplo com operador
$users = User::whereColumn('first_name', '!=', 'last_name')->get();

// Exemplo sem especificar o operador (assume igualdade)
$users = User::whereColumn('updated_at', 'created_at')->get();

```

### Combinando `where` com Closure
Permite uma lógica de consulta complexa, combinando várias condições.

```php
$users = User::where(function ($query) {
    $query->where('age', '>', 20)
          ->orWhere('name', 'like', '%John%');
})->get();
```

## Ordenação e Agregação

### `orderBy('column', 'direction')`
Ordena os resultados da consulta.

```php
$users = User::orderBy('name', 'asc')->get(); // Ordena usuários pelo nome em ordem ascendente
```

## Trabalhando com Collections

### Métodos `first`, `last`, `reverse`
Manipulam a coleção de resultados, permitindo pegar o primeiro ou último registro, ou mesmo inverter a ordem dos itens.

```php
$user = User::where('active', 1)->first(); // Primeiro usuário ativo
$lastUser = User::all()->last(); // Último usuário registrado
$reversedUsers = User::all()->reverse(); // Inverte a ordem dos usuários
```

### `toArray()`, `toJson()`
Convertem a coleção para array ou JSON, facilitando a integração com APIs ou simplesmente a manipulação dos dados.

```php
$usersArray = User::all()->toArray(); // Converte todos os usuários para array
$usersJson = User::all()->toJson(); // Converte todos os usuários para JSON
```

### `pluck`

Suponha que você tenha uma tabela de usuários (`users`) e queira obter apenas os nomes de todos os usuários. Você pode fazer isso da seguinte maneira:

```php
$names = User::pluck('name');

// Exemplo de saída
// ['Alice', 'Bob', 'Charlie']
```

Você também pode especificar uma coluna para ser usada como as chaves da coleção resultante. Por exemplo, se quiser obter os nomes dos usuários, usando seus IDs como chaves:

```php
$names = User::pluck('name', 'id');

// Exemplo de saída
// [1 => 'Alice', 2 => 'Bob', 3 => 'Charlie']
```
### `delete`
Após executar o método `delete` em um modelo, o registro correspondente é removido do banco de dados. Não há uma "saída" visível no código, mas o efeito é a remoção do registro.

```php
$user = User::find(1);
$user->delete();
// Saída: Registro do usuário com ID 1 é deletado do banco de dados.
```

### `forceDelete`
Quando o `forceDelete` é chamado, o registro é permanentemente excluído do banco de dados, ignorando o Soft Delete.

```php
$user = User::withTrashed()->find(1);
$user->forceDelete();
// Saída: Registro do usuário com ID 1 é permanentemente deletado do banco de dados.
```

### Soft Delete (`deleted_at`)
Ao usar Soft Deletes, os registros não são removidos. Em vez disso, a coluna `deleted_at` é preenchida com a timestamp atual.

```php
$user = User::find(1);
$user->delete();
// Saída na tabela: `deleted_at` da linha correspondente é atualizada para a timestamp atual.
```

### `destroy`
O método `destroy` remove um ou mais registros pelo ID.

```php
User::destroy(1);
// Saída: Registro do usuário com ID 1 é deletado do banco de dados.
```

### `fill` e `save`
`fill` preenche o modelo com dados e `save` salva no banco de dados.

```php
$user = User::find(1);
$user->fill(['name' => 'Novo Nome']);
$user->save();
// Saída no banco de dados: Nome do usuário com ID 1 é atualizado para "Novo Nome".
```

### `update`
Atualiza registros que correspondem a uma consulta.

```php
User::where('active', 1)->update(['active' => 0]);
// Saída: Todos os usuários que estavam ativos (active = 1) são atualizados para inativos (active = 0).
```

### `withTrashed`
Retorna todos os registros, incluindo os soft deleted.

```php
$users = User::withTrashed()->get();
// Saída: Coleção de todos os usuários, incluindo aqueles com a coluna `deleted_at` preenchida.
```

### `onlyTrashed`
Retorna apenas os registros soft deleted.

```php
$deletedUsers = User::onlyTrashed()->get();
// Saída: Coleção de usuários que foram soft deleted.
```

### `restore`
Restaura registros soft deleted.

```php
User::withTrashed()->where('id', 1)->restore();
// Saída: O registro do usuário com ID 1 é restaurado, `deleted_at` é definido como NULL.
```

## Controllers

- **Comando para criar**: `php artisan make:controller PrincipalController`
- **Descrição**: Cria um arquivo para o controlador com o nome `PrincipalController`. Dentro de um controlador, podemos definir métodos (ou actions) que são associados às rotas.
- **Exemplo de rota associada**:
  ```php
  Route::get('/', [PrincipalController::class, 'principal']);
  ```

Aprimorar suas anotações sobre **Views** no Laravel pode torná-las mais informativas e úteis, principalmente para quem está aprendendo a trabalhar com esse framework PHP. As **Views** são uma parte essencial do padrão MVC (Model-View-Controller) utilizado pelo Laravel, responsáveis por gerar a saída HTML que será enviada ao navegador do usuário. Vamos enriquecer suas anotações com mais detalhes e exemplos:

## Views

### Descrição
As **Views** no Laravel são arquivos responsáveis pela renderização da interface do usuário, geralmente contendo marcações HTML misturadas com PHP. Elas são usadas para apresentar dados enviados pelo Controller, seguindo o padrão MVC. O Laravel suporta várias maneiras de passar dados para as views, tornando o framework flexível e poderoso para o desenvolvimento de aplicações web.

### Localização
Por padrão, as views do Laravel são armazenadas no diretório `resources/views`. Cada view é um arquivo `.blade.php` ou `.php`, mas o uso do Blade, o motor de templates do Laravel, é recomendado por oferecer uma sintaxe mais limpa e recursos adicionais.

### Passando Dados para Views
Há diversas maneiras de passar dados do controller para uma view, permitindo flexibilidade conforme a necessidade do desenvolvedor.

#### Array Associativo
Você pode passar dados utilizando um array associativo. Este método é útil para passar múltiplos valores.

```php
public function show(int $id){
    return view('site.show', ['id' => $id]);
}
```

#### Método `compact()`
O método `compact()` do PHP é bastante útil e limpo para passar variáveis para a view. Ele cria um array associativo com nomes e valores de variáveis.

```php
public function show(int $id){
    return view('site.show', compact('id'));
}
```

#### Método `with()`
O método `with()` permite passar dados para a view encadeando chamadas para cada dado necessário. É útil para adicionar dados adicionais condicionalmente.

```php
public function show(int $id){
    return view('site.show')->with('id', $id)->with('extra', $extra);
}
```

### Blade
Blade é um poderoso motor de templates que vem com Laravel, permitindo que você escreva código PHP de forma mais limpa e legível dentro de suas views.

- **Diretivas Blade**: Blade oferece diretivas especiais para tarefas comuns, como loops e condicionais, simplificando o código PHP dentro das views.
  
  ```php
  @if ($id > 100)
      <p>O ID é maior que 100.</p>
  @endif
  ```

- **Herança de Layouts**: Blade permite a criação de layouts mestre, facilitando a reutilização de código HTML comum entre diferentes páginas.

  ```php
  {{-- Layout principal --}}
  @extends('layouts.master')

  @section('content')
      <p>Este é o conteúdo específico da página.</p>
  @endsection
  ```

- **Componentes e Slots**: Blade suporta componentes e slots, permitindo a criação de elementos de UI reutilizáveis e facilmente personalizáveis.

  ```php
  {{-- Definindo um componente --}}
  @component('components.alert')
      @slot('title')
          Aviso Importante
      @endslot

      Este é o conteúdo do alerta.
  @endcomponent
  ```
### Estruturas de Controle

#### Condicional `@if`, `@elseif`, `@else`
Blade simplifica as expressões condicionais com suas diretivas `@if`, `@elseif`, e `@else`.

```php
@if ($id > 100)
    <p>O ID é maior que 100.</p>
@elseif ($id > 50)
    <p>O ID é maior que 50, mas menor ou igual a 100.</p>
@else
    <p>O ID é 50 ou menor.</p>
@endif
```

#### Diretiva `@unless`
A diretiva `@unless` é o inverso de `@if`, executada apenas se a expressão for falsa.

```php
@unless ($id > 100)
    <p>O ID é 100 ou menor.</p>
@endunless
```

#### Loop `@for`, `@foreach`, `@while`
Blade facilita a implementação de loops com as diretivas `@for`, `@foreach` e `@while`.

```php
@foreach ($items as $item)
    <li>{{ $item }}</li>
@endforeach
```

#### Diretiva `@forelse` 
`@forelse` é uma combinação de `@foreach` e `@empty`, útil para arrays possivelmente vazios.

```php
@forelse ($items as $item)
    <li>{{ $item }}</li>
@empty
    <li>Nenhum item encontrado.</li>
@endforelse
```

#### Verificação de Existência `@isset`, `@empty`
Diretivas para verificar rapidamente se uma variável está definida ou vazia.

```php
@isset($var)
    <p>{{ $var }}</p>
@endisset

@empty($var)
    <p>Variável está vazia.</p>
@endempty
```

### Estrutura de Seleção `@switch`, `@case`
A diretiva `@switch` oferece uma maneira de executar comparações múltiplas de uma forma mais limpa.

```php
@switch($id)
    @case(1)
        Primeiro caso...
        @break

    @case(2)
        Segundo caso...
        @break

    @default
        Caso padrão...
@endswitch
```

### Herança de Template

#### Diretiva `@extends` e `@section`
Blade permite a herança de layouts, facilitando a reutilização de estruturas de páginas.

```php
@extends('layouts.master')

@section('content')
    <p>Este é o conteúdo específico da página.</p>
@endsection
```

#### Diretiva `@yield`
`@yield` é usada no layout pai para exibir o conteúdo de uma seção.

```php
<!-- No layout pai -->
<body>
    @yield('content')
</body>
```

### Proteção CSRF
O Laravel inclui proteção CSRF em seus formulários Blade automaticamente com a diretiva `@csrf`.

```php
<form method="POST" action="/profile">
    @csrf
    <!-- Campos do formulário -->
</form>
```

### Componentes e Slots
Os componentes são reutilizáveis e podem ser empacotados com lógica PHP, permitindo uma maior modularidade.

```php
@component('components.alert')
    @slot('title')
        Aviso Importante
    @endslot

    Este é o conteúdo do alerta.
@endcomponent
```

Além disso, o Laravel permite a definição de componentes Blade de uma forma mais orientada a classes, que podem ser invocados com a diretiva `@component` ou usando tags de componente, facilitando a passagem de dados e a reutilização de lógica de visualização em diferentes partes da aplicação.

## Routes
- **Parâmetros Opcionais**: Utilize `?` para indicar que um parâmetro é opcional. 
- **Restrições com Regex**: É possível restringir os parâmetros das rotas usando expressões regulares.
- **Métodos comuns**: `get`, `post`, `put`, `delete`.
- **Exemplo com parâmetro opcional, regex e callback**:
  ```php
  Route::get('/contact/{nome}/{id?}', function (string $nome, int $id = 1) {
      echo "Estamos aqui ".$nome." ".$id;
  })->where('id', '[0-9]+');
  ```
- **Uso de Prefixo**: Agrupar rotas que compartilham um prefixo comum.
  ```php
  Route::prefix('/app')->group(function () {
      Route::get('/clientes', function(){ return 'Clientes'; });
      Route::get('/fornecedores', function(){ return 'Fornecedores'; });
      Route::get('/produtos', function(){ return 'Produtos'; });  
  });
  ```
- **Nomeando Rotas**: Facilita a referência interna às rotas.
  ```php
  Route::get('/rota1', function () {
      echo "Teste redirect";
  })->name('site.rota1');
  ```

## Redirecionamentos

- **Exemplo de Redirect**:
  ```php
  Route::get('/rota2', function () {
      return redirect()->route('site.rota1');  // Redireciona para a rota site.rota1.
  })->name('site.rota2');
  ```

## Rota para Erros

- **Fallback**: Define uma rota para ser chamada quando nenhuma outra rota corresponde.
  ```php
  Route::fallback(function(){
      echo "Aqui não temos o que procurar...";
  });
  ```

## Passando Parâmetros para Métodos do Controller

- **Exemplo**:
  Se existe a rota:
  ```php
  Route::get('/teste/{id}', [TesteController::class, 'teste'])->name('teste');
  ```
  No método `teste` do `TesteController`, o parâmetro `id` deve ser recebido assim:
  ```php
  public function teste(int $id){
      echo "O id é: ".$id;
  }
  ```
  A ordem dos parâmetros na rota deve ser seguida na função do controlador.


---

