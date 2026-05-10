# 🎬 Cinemati

> Asistente de Telegram con AI que ayuda a grupos de amigos a decidir qué ver juntos. Cruza wishlists personales, contexto del grupo y matchmaking inteligente con votación secreta y revelación dramática.

## 🎯 El problema

"¿Qué miramos esta noche?" es una de las preguntas más dolorosas del viernes a la noche en cualquier grupo de amigos. Las plataformas de streaming recomiendan en base a lo que ya viste, pero no resuelven el problema real: **decidir junto con otra gente que tiene gustos distintos al tuyo, en un momento específico, con un estado de ánimo específico**.

Cinemati es un anti-Netflix: no te recomienda algo a vos, ayuda a tu grupo a decidir.

## ✨ Cómo funciona

1. **Cada amigo se registra** con `/registro` y carga sus géneros favoritos, vetos y preferencias.
2. **Mantienen wishlists personales** sumando pelis y series con `/agregar`. El sistema enriquece cada item con metadata real de TMDB (duración, géneros, plataformas, rating).
3. **Cuando el grupo se junta**, alguien arranca `/sesion` indicando tiempo disponible y "vibe" del grupo.
4. **Claude actúa como matchmaker**: cruza las wishlists, considera el contexto, y propone 3 candidatos con argumentos personalizados para cada participante.
5. **Votación secreta**: cada uno recibe los candidatos por privado y vota sin que los demás vean.
6. **Revelación dramática** en el grupo: cuenta regresiva, anuncio del ganador, link directo. Si hay empate, Claude desempata con criterio justificado (no random).
7. **Feedback al día siguiente**: el sistema aprende qué tan buena fue la elección.

## 🏗️ Arquitectura

El sistema usa un **router pattern**: un workflow principal recibe todos los mensajes del bot y delega a sub-workflows especializados según el comando.

```
Telegram → Router → [Switch por comando] → Sub-workflow específico
                                          ↓
                              Sheets / TMDB / Claude API
                                          ↓
                                     Telegram (respuesta)
```

Cada comando es un sub-workflow independiente, lo que permite desarrollarlos y testearlos en aislamiento.

## 🛠️ Stack técnico

- **n8n** (self-hosted con Docker en VPS propia) — orquestador de workflows
- **Telegram Bot API** — interfaz de usuario conversacional
- **Google Sheets** — base de datos liviana con 6 pestañas relacionadas
- **TMDB API** — metadata de pelis y series (gratis, oficial)
- **Anthropic Claude API** — parsing de texto libre, matchmaking, desempate justificado, síntesis de patrones

## 📊 Modelo de datos

| Pestaña | Propósito |
|---|---|
| `usuarios` | Registro maestro de quién está en el sistema |
| `wishlists` | Pelis/series que cada usuario quiere ver, con metadata enriquecida |
| `sesiones` | Registro de cada noche grupal: participantes, contexto, candidatos, ganador |
| `votos` | Votos individuales (append-only para auditoría) |
| `feedback` | Calificaciones post-sesión para aprendizaje continuo |
| `_log` | Eventos del sistema para debugging y observabilidad |

Las relaciones se mantienen vía IDs únicos generados por el sistema (no por nombres ni IDs internos de las herramientas).

## 🚧 Estado del proyecto

Proyecto en construcción activa. Este README va a evolucionar a medida que avancen los módulos.

**Completado:**
- Diseño del schema de base de datos
- Bot de Telegram operativo (`@CinematiBot`)
- Workflow Router con manejo de 5 comandos + fallback elegante
- **Sub-workflow `Cinemati - Registro` (publicado v2.1)**
  - Onboarding limpio para usuarios nuevos
  - Validación con AI de mensajes en formato libre
  - Detección de duplicados contra Google Sheets
  - Manejo de 4 escenarios distintos con mensajes específicos

**En desarrollo:**
- Sub-workflow `Cinemati - Agregar` (con integración TMDB)
- Sub-workflow `Cinemati - Sesión` (matchmaking con Claude)
- Sub-workflow `Cinemati - Votación` (manejo de estado asíncrono)
- Sub-workflow `Cinemati - Feedback`

## 🧠 Decisiones de diseño

Algunas decisiones técnicas que vale la pena documentar:

- **Mensaje único estructurado** para `/registro` en vez de conversación paso a paso. Razón: aprovecha la capacidad de Claude para parsear texto libre y reduce la fricción del usuario. Una sola pegada de mensaje en vez de múltiples idas y vueltas.

- **Router pattern con sub-workflows** en vez de un workflow monolítico. Razón: cada comando se desarrolla y testea en aislamiento, y la complejidad no escala con la cantidad de comandos.

- **IDs propios del sistema** en vez de IDs internos de Sheets/Notion. Razón: independencia de la plataforma de almacenamiento. Si mañana se migra a una base de datos real, los IDs siguen siendo válidos.

- **Voto secreto + revelación dramática** como ritual grupal. Razón: convierte el sistema de "recommender system" a "experiencia social diseñada". El elemento de juego aumenta la adopción y hace al producto memorable.

## 📜 Licencia

[MIT](LICENSE) — usalo, modificalo, adaptalo. Solo mantené el atribución.

## 👤 Autor

Construido por [Franco Barassi](https://github.com/franbarassi) como proyecto de aprendizaje en automatización con AI usando n8n.
