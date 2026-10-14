---
name: ejecuta-plan-por-etapas
description: Usar cuando el usuario quiera ejecutar un plan ya elaborado. Dispara con "ejecuta", "hazlo", "sigue".
---

# Ejecuta plan por etapas

Ejecutar o empezar un plan previamente generado. El usuario te indicará el plan o preguntale si hay alguno. Hazlo por las etapas que fueron concebidas. Antes de seguir con la siguiente etapa, pide confirmación.

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

      within ".test-suite-run-list" do
        expect(page).to have_content("Running")
      end

      task.update!(exit_code: 0)

      within ".test-suite-run-list" do
        expect(page).to have_content("Passed")
      end
    end
  end
end
```