# HomeAssistant Taryfy energii elektrycznej

Pola z cenami i bieżącą taryfą
(configuration.yaml)

```
input_number:
  cena_energii_szczyt_poranny:
    name: Cena energii szczyt przedpołudniowy za kWh
    min: 0
    max: 2
    step: 0.01
  cena_energii_szczyt_popoludniowy:
    name: Cena energii szczyt popołudniowy za kWh
    min: 0.4
    max: 1.4
    step: 0.01
  cena_energii_poza_szczytem:
    name: Cena enregii poza szczytem za kWh
    min: 0.4
    max: 1.4
    step: 0.01
  cena_energii_plaska:
    name: Cena energii płaska za kWh
    min: 0.4
    max: 1.4
    step: 0.01
  taryfa_energii:
    name: Bieżąca taryfa za energię elektryczną
    min: 1
    max: 3
    step: 1
```

Encje z ceną
(configuration.yaml)
```
# Cena energii elektrycznej C11
  - platform: template
    sensors:
      cena_energii_c11:
        device_class: monetary
        friendly_name: Cena energii C11
        unit_of_measurement: "PLN/kWh"
        value_template: >
          {% set cena_energii_c11 = states('input_number.cena_energii_plaska') %}
          {{ cena_energii_c11 }}

# Cena energii elektrycznej B23
  - platform: template
    sensors:
      cena_energii_b23:
        device_class: monetary
        friendly_name: Cena energii B23
        unit_of_measurement: "PLN/kWh"
        value_template: >
          {% set taryfa_energii = states('input_number.taryfa_energii') | round(0,0) %}
          {% set cena_energii_b23_1 = states('input_number.cena_energii_szczyt_poranny') %}
          {% set cena_energii_b23_2 = states('input_number.cena_energii_szczyt_popoludniowy') %}
          {% set cena_energii_b23_3 = states('input_number.cena_energii_poza_szczytem') %}
          {% if taryfa_energii == 1 %}
            {{ cena_energii_b23_1 }}
          {% elif taryfa_energii == 2 %}
            {{ cena_energii_b23_2 }}
          {% else %}
            {{ cena_energii_b23_3 }}
          {% endif %}
```


Autmatyzacja przełączania taryfy aby encja 'cena_energii_b23' była wypełniona aktualną ceną energii
(automations.yaml)
```
- id: '1644404621487'
  alias: Ustaw taryfę energii B23
  description: Ustawia bieżącą cenę energii
  trigger:
  - platform: time
    at: 07:00
  - platform: time
    at: '13:00'
  - platform: time
    at: '16:00'
  - platform: time
    at: '19:00'
  - platform: time
    at: '21:00'
  - platform: time
    at: '22:00'
  condition: []
  action:
  - choose:
    - conditions:
      - condition: time
        weekday:
        - sat
        - sun
        after: 00:00
        before: '23:59'
      sequence:
      - service: input_number.set_value
        data:
          value: 3
        target:
          entity_id: input_number.taryfa_energii
    - conditions:
      - condition: time
        after: 07:00
        before: '12:59'
        weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
      sequence:
      - service: input_number.set_value
        data:
          value: 1
        target:
          entity_id: input_number.taryfa_energii
    - conditions:
      - condition: template
        value_template: '{{ now().month >= 10 or now().month <= 3 }}'
      - condition: time
        weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
        after: '16:00'
        before: '20:59'
      sequence:
      - service: input_number.set_value
        data:
          value: 2
        target:
          entity_id: input_number.taryfa_energii
    - conditions:
      - condition: template
        value_template: '{{ now().month >= 4 or now().month <= 9 }}'
      - condition: time
        after: '19:00'
        before: '21:59'
        weekday:
        - mon
        - tue
        - wed
        - thu
        - fri
      sequence:
      - service: input_number.set_value
        data:
          value: 2
        target:
          entity_id: input_number.taryfa_energii
    default:
    - service: input_number.set_value
      data:
        value: 3
      target:
        entity_id: input_number.taryfa_energii
  mode: single
  ```
  
