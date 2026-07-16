# GRADING TEST — Corrección automatizada de exámenes manuscritos

Sistema de corrección de exámenes escritos a mano, desarrollado y probado con el examen
final de Álgebra Lineal MAT1224 (2026-1). No consiste en que "un LLM corrige": el modelo
de lenguaje propone, la matemática verifica (SymPy) y el humano decide los casos dudosos.
Cada pregunta termina en una banda AUTO o en una cola de revisión humana según un score
de confiabilidad calculado a partir de señales medibles; cuando el sistema no puede leer
o calificar con garantías, lo declara en lugar de entregar una nota de todos modos.

El documento de referencia es el blueprint (3–4 páginas): qué problemas del enfoque
directo resuelve, el pipeline de 7 fases, las bandas de revisión, los resultados del
piloto de calibración y cómo replicarlo en otro examen.

- [`index.html`](index.html) — versión web (servida con GitHub Pages).
- [`blueprint_correccion.md`](blueprint_correccion.md) — el mismo documento, legible
  directamente en GitHub.

Estado (julio 2026): 7 fases implementadas y operativas, 76 tests automatizados, piloto
de calibración en curso contra corrección humana.

Este repositorio difunde el diseño del sistema. El código del pipeline, la rúbrica de
referencia y los reportes del piloto no están publicados aquí; están disponibles a
pedido.
