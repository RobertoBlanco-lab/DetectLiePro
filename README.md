# DetectLiePro
“Motor de análisis de voz con fusión variable-por-variable y capa ML ligera”
# DetectLie Pro — Motor de fusión variable-por-variable
**Autor:** Roberto Blanco Gómez  

## Qué es
Motor de análisis de voz que estima consistencia vs. inconsistencia respecto al patrón del usuario.  
Combina un análisis variable-por-variable (jitter, shimmer, pausas, energía, HNR/SNR, etc.) con una capa ML ligera para refinar decisiones cuando hay suficiente señal.  

## Cómo funciona (resumen técnico entendible)
- **Warm-up por usuario**: primeras N tomas (p. ej., 6) se usan para calibrar medias y desviaciones (EMA), sin emitir juicios fuertes.  
- **z-scores robustos + clipping**: cada variable se normaliza con su media/std y se acota ±zClip para evitar outliers.  
- **Evidencia por variable**:  
  - Umbral τ define si el desvío es significativo.  
  - Se calculan *eLie* (inconsistencia) y *eTruth* (consistencia) en [0,1].  
  - Se agregan en un `pTruthRaw = eTruth / (eTruth + eLie)`.  
- **Reglas aprendidas opcionales**: direcciones por variable (*signLie*) pueden estimarse desde histórico etiquetado (CSV/TSV) para personalizar la sensibilidad.  
- **Quality gates**: si el SNR es bajo o HNR inestable, se de-priorizan o ignoran esas variables, reduciendo falsos positivos por mala calidad de audio.  
- **Penalización/realce**: si muchas variables apuntan fuerte en el mismo sentido se aplica un cap/floor (evita sesgos de un único indicador).  
- **Mezcla con ML (blend)**: logística online (SGD + L2) o NN externa. Se activa solo con suficientes ejemplos y precisión mínima; si no, el motor por-variable manda.  
- **Estabilidad temporal**: pTruth se suaviza con ventana corta para una UI más estable (menos parpadeo).  
- **DCI (Decision Clarity Index)**: índice 0–100 que combina fuerza de evidencia, margen vs umbral, acuerdo entre variables, estabilidad temporal y calidad de audio.  

## Interacción y flujo de app
1. Grabar → Analizar → Mostrar % de consistencia y explicación breve (variables más influyentes).  
2. Botones de feedback (“Fue Verdad / Fue Mentira”): entrenan solo la capa ML (evita drift en el motor base).  
3. Nuevo usuario / Reset: doble confirmación para evitar toques accidentales; borra calibración por-usuario y estado ML.  

## Métricas y parámetros relevantes (por defecto seguros)
- `thresholdUsuario ≈ 0.65` (decisión conservadora).  
- `zClip ≈ 3.0`, `anomalyZThreshold ≈ 2.0`.  
- `mlBlendLambda ≈ 0.20`, `mlAccGate ≈ 0.50`, `mlMin ≈ 4`.  
- SNR mínimo recomendado ≈ 0.15 para decisiones; HNR con alta desviación reduce su peso.  

## Entrenamiento desde histórico (opcional)
- Importar CSV/TSV etiquetado → aprender *signLie*, correlaciones y fiabilidad por variable.  
- El modelo aprendido no toca pesos globales: solo guía direcciones/umbrales por variable, manteniendo estabilidad.  

## Buenas prácticas de uso
- Usar el mismo micro/entorno cuando sea posible.  
- Evitar ruidos intensos; si el SNR baja, los resultados se degradan.  
- Etiquetar unas 50–100 tomas variadas mejora claramente la capa ML.  
- Interpretar el resultado como consistencia con tu patrón, no como “detector de mentiras” absoluto.  

## Aviso importante
Este software **no es un instrumento médico, legal ni forense**.  
Es un analizador de patrones acústicos.  
No usar en contextos de alto riesgo o donde se requiera evidencia pericial.  

---

## Reflexión personal
El prototipo está quedando finísimo, la tasa de acierto mejora y la viabilidad del proyecto se hace patente.  

Este verano me recuerda a un año en el que suspendí una asignatura y tuve que prepararla para septiembre. Al centrarme solo en ella terminé adentrándome más de lo esperado, y al año siguiente aprobarla fue mucho más fácil.  

Con esta app ha pasado lo mismo: ensayo, error, empezar varias veces desde cero, reiniciar la forma de encaminar el proyecto… Ha sido mi **puzzle de verano de más piezas de las esperadas**: más de 2000 líneas de código para el resultado final.  

El resultado, desde mi punto de vista, es incluso más fiable de lo que pretendía inicialmente.
