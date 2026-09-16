---
name: clima-local
description: Consulta el clima actual y el pronóstico de una ubicación local (por defecto Zapopan Norte, Jalisco, México) y reporta si va a llover, estará nublado o soleado. Usar cuando el usuario pida revisar, checar o saber el clima de su zona.
---

# Clima local

Este skill obtiene el clima actual y el pronóstico de una ubicación para reportar de forma clara
si va a llover, si estará nublado o si estará soleado.

## Ubicación por defecto

Si el usuario no especifica una ubicación distinta, usa: **Zapopan Norte, Jalisco, México**.

## Pasos

1. Toma la ubicación indicada en `args` (o la ubicación por defecto si no se especifica ninguna).
2. Usa la herramienta `WebSearch` con una consulta como:
   `clima <ubicación> hoy pronóstico por hora`
3. De los resultados, extrae:
   - Probabilidad de lluvia (%)
   - Condición general (lluvia, nublado, soleado, parcialmente nublado)
   - Temperatura máxima y mínima
   - Viento (si está disponible)
4. Reporta la conclusión en una frase directa: si va a llover, si estará nublado o soleado,
   usando la probabilidad de lluvia como criterio (p. ej. ≥50% → "probable que llueva").
5. Incluye siempre la sección de fuentes (enlaces) que devuelve `WebSearch`, en formato markdown.

## Formato de salida

Responde en español, de forma breve:

- Condición principal (lluvia / nublado / soleado)
- Probabilidad de lluvia
- Temperatura (min/max)
- Fuentes usadas
