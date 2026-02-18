## Change Log

| Fecha       | Versión | Autor         | Descripción                                |
|-------------|---------|---------------|--------------------------------------------|
| 2026-02-13  | 1.0     | Carlos Galan  | ADR original sobre librería de fuzzy match |

---

## Referenced Use Case(s)

- Validación y sugerencia automática de nombres de cámaras en el sistema mediante comparación aproximada de cadenas contra un catálogo predefinido.

---

## Context

Actualmente, el sistema valida el nombre de la cámara comparando de manera estricta la cadena ingresada con un listado de opciones válidas. El objetivo es mejorar esa experiencia, dando además sugerencias inteligentes de nombres similares cuando el dato no coincida exactamente.  
La recomendación inicial fue el uso de la librería `rapidfuzz` de Python. Sin embargo, el proyecto está desarrollado en .NET, por lo que se evaluaron alternativas que pudieran integrarse de manera nativa, buscando minimizar la complejidad operativa y de mantenimiento.

El análisis comparativo incluyó estas librerías:
- rapidfuzz (Python) [referencia]
- fuzzywuzzy (Python)
- FuzzySharp (.NET)
- Raffinert.FuzzySharp (.NET)

Se evaluaron precisión (vs. rapidfuzz), calidad de sugerencia, mantenimiento del proyecto y facilidad de integración.

---

## Proposed Design

Adoptar la librería **Raffinert.FuzzySharp** para realizar la comparación aproximada de cadenas en el módulo de validación de nombres de cámara dentro de la aplicación .NET. Esta librería será la encargada de comparar el nombre ingresado y sugerir la coincidencia más cercana a partir del catálogo definido.

---

## Considerations

- **Compatibilidad:** Emplear una librería nativa de .NET garantiza integración sencilla y soporte futuro más claro.
- **Precisión:** En una muestra de 2200 nombres, Raffinert.FuzzySharp coincidió con rapidfuzz en aproximadamente 1525 casos, presentando cerca de 675 discrepancias (equivalente a las otras librerías .NET o Python). No obstante, la calidad subjetiva de las sugerencias de Raffinert.FuzzySharp resultó superior.
- **Mantenimiento:** Raffinert.FuzzySharp tiene actividad reciente en GitHub. FuzzySharp (alternativa .NET) no tiene actualizaciones en varios años.
- **Facilidad de integración:** Usar librerías Python como rapidfuzz o fuzzywuzzy requeriría puentes tecnológicos, complicando el despliegue y mantenimiento.
- **Satisfacción del cliente:** Aunque rapidfuzz ofrecía buenos resultados, la alternativa .NET seleccionada cumple con los objetivos de funcionalidad y calidad percibida.

---

## Decision

**Se selecciona la librería Raffinert.FuzzySharp (.NET) como solución para la comparación aproximada de cadenas y sugerencias de nombres de cámaras.**

- Se prioriza la integración nativa y el soporte de largo plazo en .NET.
- La calidad de las sugerencias es considerada adecuada de acuerdo a pruebas y revisión del equipo.
- Se descartó la integración de mecanismos Python por complejidad, y FuzzySharp por falta de soporte.

---

## References

- [Raffinert.FuzzySharp en GitHub](https://github.com/raffinert/FuzzySharp)
- [FuzzySharp en GitHub](https://github.com/JakeBayer/FuzzySharp)
- [rapidfuzz en GitHub](https://github.com/rapidfuzz/RapidFuzz)
- [fuzzywuzzy en GitHub](https://github.com/seatgeek/fuzzywuzzy)
- [Excel comparativo con los casos de prueba reales](https://applaudostudios.sharepoint.com/:x:/r/sites/DollarcityTeam/Shared%20Documents/05%20-%20Mejoras%20DCORI/ADR%20Docs/RapidFuzzy%20alternatives.xlsx?d=w64622d7b51284f1c9b045e4cc6ec4029&csf=1&web=1&e=XzxUeV)
