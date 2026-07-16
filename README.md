GRADING TEST — Corrección automatizada de exámenes manuscritos

Pipeline de corrección de exámenes escritos a mano (caso experimental: examen final de
Álgebra Lineal MAT1224, 120 pts, 15 ítems). No consiste en que "un LLM corrige": el LLM
propone, la matemática verifica (SymPy) y el humano decide los casos dudosos. Cuando el
sistema no puede leer o calificar con garantías, lo declara y deriva el caso a revisión
humana en lugar de entregar una nota de todos modos.

La decisión es por pregunta, no por examen: cada pregunta termina en una de tres bandas
según su score de confiabilidad (1–10) — AUTO (score ≥ 9, nota aceptada sin revisión
individual, con auditoría aleatoria del 10 %), revisión rápida (7–8, el corrector
verifica solo los puntos señalados) o revisión completa (< 7, corrección manual con la
transcripción como apoyo). Una pregunta con baja confianza no bloquea a las demás.

Estado (julio 2026): 7 fases implementadas y operativas, 76 tests automatizados en
verde, piloto de calibración en curso contra exámenes corregidos a mano.

Documentación


docs/blueprint_correccion.md — descripción del modelo
(3–4 páginas): qué problemas del enfoque directo resuelve, el pipeline, las bandas, los
resultados del piloto y cómo replicarlo en otro examen. También en versión
HTML autocontenida. Es el documento para empezar.
docs/flujo_grading_algebra_lineal.md — diseño
técnico completo (fases, scores, códigos de excepción E1–E14, stack).
docs/pipeline.md — diagrama operativo rápido.


Cómo funciona

escaneo (300 dpi) → F0 preproceso y segmentación por contenido (OpenCV + Vision)
                  → F1 transcripción (Claude Vision + segunda lectura Gemini + Tesseract)
                  → F2 reconciliación entre lecturas → conf_lectura
                  → F3 corrección por ítem contra la rúbrica JSON (Claude)
                  → F4 verificación simbólica (SymPy)
                  → F5 score de confiabilidad (6 señales, 1–10)
                  → F6 reportes por estudiante (.md y .html) y colas AUTO / revisión

Puntos de diseño principales:


Rúbrica cerrada (00_pauta/rubrica_examen_final.json): el modelo solo puede aplicar
las penalizaciones listadas, con su valor exacto; las notas globales fijan el
tratamiento de soluciones alternativas y de errores arrastrados (se penalizan una vez).
Para correr otro examen se cambia este archivo, no el código.
Lecturas redundantes: dos transcripciones independientes por modelos de proveedores
distintos (Claude y Gemini); la segunda lectura solo alimenta la medida de confianza,
nunca aporta contenido a la nota.
Verificación objetiva: SymPy ejecuta los snippets de la rúbrica y fija los
resultados calculables (9 de 15 ítems); una contradicción con el modelo se resuelve a
favor de SymPy.
Trazabilidad: cada punto otorgado o descontado cita evidencia textual de la
transcripción; 14 códigos de excepción documentan por qué algo fue a revisión.
Orquestación reanudable: estado por estudiante-ítem en SQLite; un error de API a
mitad del lote no pierde el trabajo hecho. Los componentes opcionales (Tesseract,
segunda lectura) no detienen el pipeline si faltan: la señal ausente queda registrada y
el score lo refleja.


Estructura del repositorio

Las carpetas numeradas siguen el recorrido de un examen: los datos entran por
01_escaneos/ y salen por 05_salidas/.

GRADING AI/
├── docs/                  Blueprint, diseño técnico y contexto del proyecto
├── 00_pauta/              Rúbrica JSON + enunciado + solucionario (se versiona)
├── 01_escaneos/           [INPUT] PDF crudos del escáner, ≥300 dpi, uno por estudiante
├── 02_preprocesado/       F0 · imágenes limpias (deskew, CLAHE, binarizado) + recortes
├── 03_transcripciones/    F1–F2 · lecturas por fuente + reconciliación + conf_lectura
├── 04_grading/            F3–F5 · corrección por ítem + verificación SymPy + score
├── 05_salidas/            [OUTPUT] F6 · reporte .md/.html por estudiante + colas
├── 06_estado/             Orquestación: SQLite reanudable + logs
├── 07_calibracion/        Piloto: exámenes corregidos a mano + métricas auto vs. humano
├── scripts/               Código del pipeline, una subcarpeta por fase + tests
├── requirements.txt       Dependencias
└── .gitignore             Excluye datos de estudiantes y secretos

Puesta en marcha

bashpip install -r requirements.txt
# Tesseract es un binario del sistema (opcional), aparte de pytesseract:
#   Windows: instalar Tesseract-OCR y añadirlo al PATH


Claves de API en .env (Anthropic; la de Google es opcional, habilita la segunda
lectura).
Escaneos a 300 dpi en 01_escaneos/, un archivo por estudiante.
Ejecutar la orquestación: python -m scripts.orquestacion --vision. La corrida es
reanudable y permite re-correr estudiantes puntuales.
Los reportes quedan en 05_salidas/, con las fichas por pregunta en las colas
auto/, revision_rapida/ y revision_completa/.


Los tests están en scripts/tests/ (pytest).

Calibración

Antes de usar el sistema en un examen nuevo: corregir a mano 3–5 exámenes, correr el
pipeline sobre los mismos y comparar. Los desajustes se corrigen en los desgloses de la
rúbrica, no en las instrucciones al modelo. En el piloto de MAT1224 este ciclo llevó el
acuerdo exacto por ítem de 45 % a 10 de 15 ítems, con la pregunta 1 completa en banda
AUTO coincidiendo con el corrector humano; los deltas restantes responden a la política
del curso sobre resultados sin justificación. Meta del piloto completo (15–20 exámenes):
≥98 % de acuerdo en la banda AUTO. Detalle en el blueprint, §5.

Privacidad

01_escaneos/, 02_preprocesado/, 03_transcripciones/, 04_grading/, 05_salidas/,
06_estado/ y 07_calibracion/ contienen datos de estudiantes (exámenes, nombres, RUT,
notas). El .gitignore los excluye del control de versiones: se versiona el código y la
rúbrica, no los datos personales. Usar los niveles pagados de las APIs (los gratuitos
pueden usar los datos para entrenamiento).
