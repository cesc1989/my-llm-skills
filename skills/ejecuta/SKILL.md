---
name: ejecuta-plan-por-etapas
description: Usar cuando el usuario quiera ejecutar un plan ya elaborado. Dispara con "ejecuta", "hazlo", "sigue".
---

# Ejecuta plan por etapas

Ejecutar o empezar un plan previamente generado. El usuario te indicará el plan o preguntale si hay alguno. Hazlo por las etapas que fueron concebidas. Pide confirmación antes de seguir con la siguiente etapa.

## Consideraciones importantes

- Manten un buen espaciado en código Ruby y ERB.

Ejemplos:

Mal espaciado en código Ruby:
```ruby
def example_function
  someobject = SomeObject.new
  someobject.do_something
end
```

Buen espaciado en código Ruby:
```ruby
def example_function
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

- En ERB y HTML, además de la línea en blanco entre elementos hermanos, sigue estas reglas. El objetivo es que quien revise el código vea el HTML menos apiñado.
  1. Sin línea en blanco justo después de abrir una etiqueta: el primer hijo va pegado a su padre.
  2. Si una etiqueta tiene los atributos en varias líneas, el `>` de cierre va solo en su línea, alineado con el inicio de la etiqueta.
  3. El contenido de un elemento (texto o `<%= %>`) va en su propia línea, indentado. Nunca `<div><%= algo %></div>` en una sola línea.

Mal espaciado en código ERB:
```erb
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
```erb
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

Como el texto queda con saltos de línea alrededor, en las pruebas que comparen el texto de un elemento usa `.text.strip` o `include` en vez de comparar el texto exacto.

- En Ruby, siempre envuelve los argumentos de una llamada a método en paréntesis, también en pruebas (`get`, `patch`, `post`, `visit`, `within`, etc.).

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

- Cuando una llamada o un hash no cabe cómodo en una línea, pon cada argumento o atributo en su propia línea. No agrupes varios atributos por línea.

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

- Si escribes pruebas en Minitest o RSpec, los examples deben ser autocontenidos, es decir, la preparación de datos, ejecucíon y verificación del resultado, y si lo hay, la limpieza, deben pasar en el mismo bloque `it`.

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