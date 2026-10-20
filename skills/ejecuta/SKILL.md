---
name: ejecuta-plan-por-etapas
description: Usar cuando el usuario quiera ejecutar un plan ya elaborado. Dispara con "ejecuta", "hazlo", "sigue".
---

# Ejecuta plan por etapas

Ejecutar o empezar un plan previamente generado. El usuario te indicará el plan o preguntale si hay alguno. Hazlo por las etapas que fueron concebidas. Pide confirmación antes de seguir con la siguiente etapa.

## Consideraciones técnicas importantes

Manten un buen espaciado y alineado en código Ruby y ERB.

### Espaciado

Me gusta leer código bien espaciado. Espaciado en bloques de códigos significativo y que estén relacionados "gramaticalmente". Con esto voy a que me gusta ver un espacio entre asignación de variables y ejecución de funciones/métodos.

Atención a los ejemplos a continuación. No son exhaustivos pero sí representativos.

Mal espaciado en código Ruby:
```ruby
def example_function
  a_variable = 1
  someobject = SomeObject.new
  someobject.do_something
end
```

Buen espaciado en código Ruby:
```ruby
def example_function
  a_variable = 1
  someobject = SomeObject.new

  someobject.do_something
end
```

Mal espaciado en código ERB:
```html
<table>
  <th></th>
  <tbody></tbody>
  <tfooter></tfooter>
</table>
```

Buen espaciado en código ERB:
```html
<table>
  <th></th>

  <tbody></tbody>

  <tfooter></tfooter>
</table>
```

### Alineado

En ERB y HTML, además de la línea en blanco entre elementos hermanos, sigue estas reglas. El objetivo es que quien revise el código vea el HTML menos apiñado.

1. Sin línea en blanco justo después de abrir una etiqueta: el primer hijo va pegado a su padre.
2. Si una etiqueta tiene los atributos en varias líneas, el `>` de cierre va solo en su línea, alineado con el inicio de la etiqueta.
3. El contenido de un elemento (texto o `<%= %>`) va en su propia línea, indentado. Nunca `<div><%= algo %></div>` en una sola línea.

Mal espaciado en código ERB:
```html
<div class="menu-link"
     data-controller="clipboard"
     data-clipboard-text-value="<%= menu_url %>">

  <div class="menu-link-title"><%= t(".title") %></div>
  <div class="menu-link-url">
    <span class="menu-link-host"><%= host %></span>
    <strong><%= path %></strong>
  </div>
</div>
```

Buen espaciado en código ERB:
```html
<div class="menu-link"
     data-controller="clipboard"
     data-clipboard-text-value="<%= menu_url %>"
>
  <div class="menu-link-title">
    <%= t(".title") %>
  </div>

  <div class="menu-link-url">
    <span class="menu-link-host">
      <%= host %>
    </span>

    <strong>
      <%= path %>
    </strong>
  </div>
</div>
```

> Nota: Como el texto queda con saltos de línea alrededor, en las pruebas que comparen el texto de un elemento usa `.text.strip` o `include` en vez de comparar el texto exacto.

#### Excepciones

Si una etiqueta HTML que no se puede cerrar sola no lleva contenido, no alineas la etiqueta que cierre. Cierra en la misma linea.

Mal:
```html
<div class="padre">
  <i span="fa fas-icon">
  </i>
</div>
```

Bien (excepción):
```html
<div class="padre">
  <i span="fa fas-icon"></i>
</div>
```

### Uso de paréntesis

En Ruby, siempre envuelve los parámetros y argumentos de un método en paréntesis. En la definición y en las pruebas (ej: `get`, `patch`, `post`, `visit`, `within`, etc.).

Mal:
```ruby
patch "/squad/#{player_id}/release"
within ".sidebar" do
```

Bien:
```ruby
patch("/squad/#{player_id}/release")
within(".sidebar") do
```

### Alineación de argumentos

Cuando una llamada o un hash no cabe cómodo en una línea, pon cada argumento o atributo en su propia línea. No agrupes varios atributos por línea.

Mal:
```ruby
player_id = players.insert(
  name: "Jugador", age: "25", overall: "70", role: 0,
  height: "180", club_id: club_id
)
```

Bien:
```ruby
player_id = players.insert(
  name: "Jugador",
  age: "25",
  overall: "70",
  role: 0,
  height: "180",
  club_id: club_id
)
```

### Patrón de prueba en 4 fases (Four-phase test de Thoughtbot](https://thoughtbot.com/blog/four-phase-test) )

Si escribes pruebas en Minitest o RSpec, los examples deben ser autocontenidos, es decir, la preparación de datos, ejecucíon y verificación del resultado, y si lo hay, la limpieza, deben pasar en el mismo bloque `it`.

Ejemplo de una prueba con este patrón:
```ruby
describe "Sidebar test suite run status", type: :system do
  context "when a test suite run finishes" do
    it "shows the finished status in the sidebar" do
      task = create(:task, :dispatched)
      test_suite_run = task.test_suite_run
      repository = test_suite_run.repository

      test_suite_run.cache_status
      allow_any_instance_of(User).to receive(:can_access_repository?).and_return(true)
      login_as(repository.user)

      visit repository_test_suite_run_path(repository, test_suite_run)

      within(".test-suite-run-list") do
        expect(page).to have_content("Running")
      end

      task.update!(exit_code: 0)

      within(".test-suite-run-list") do
        expect(page).to have_content("Passed")
      end
    end
  end
end
```

> Nota: no pongas comentarios que separen cada etapa del test. Eso es innecesario. Solo pon comentarios que explique particularidades del test.
