# CUESTIONARIO-V2

Cuestionario clínico v2 para Centro NextHorizont Health.

## Características

- **8 pasos** (95+ campos clínicos):
  - Steps 1-5: FREE — datos básicos, salud, operaciones, antecedentes, estilo de vida (33 campos, idénticos a v1)
  - Step 6: gate de código premium (formato `NH-XXXX-XXXX`)
  - Step 7: PREMIUM — analíticas, comorbilidades graves, medicación detallada, estilo de vida ampliado (52 campos)
- Plan FREE: el paciente puede saltar Step 6 y enviar solo los campos básicos
- Plan PREMIUM: requiere canjear un código válido (validado contra `nm_premium_codes` en Supabase)
- Genera informe orientativo en 4 estados (verde/amarillo/rojo/incompleto)
- Deploy automático en Vercel

## Subdominio

`cuestionario.nexthorizont.com`

## Flujo

```
Landing centro.nexthorizont.com
   │ Click "Evaluación gratis"
   ▼
Edge Function /api/registro
   │ INSERT clinica_enlaces_pacientes (token)
   ▼
cuestionario.nexthorizont.com/?token=XXX
   │ Steps 1→5 (FREE)
   │ Step 6 (premium gate, opcional)
   │ Step 7 (PREMIUM, solo si código válido)
   │ Submit
   ▼
n8n /webhook/cuestionario-v2
   │ INSERT clinica_respuestas_pacientes (33 ó 85 columnas)
   │ Si premium: RPC consume_premium_code()
   │ Genera informe (motor mapper+engine+translator)
   │ INSERT nm_informes_paciente
   │ Telegram a Sergio
   ▼
centro.nexthorizont.com/informe/<token-informe>
   │ Página renderiza el informe (HITO 4.F)
```

## Stack

- HTML + JS estático (servido como Static Site)
- Backend: workflow n8n + Supabase
- Visualización informe: centro.nexthorizont.com/informe/:token

## Diferencias respecto al v1

| Aspecto | v1 (formulario1.nexthorizont.ai) | v2 (este repo) |
|---------|----------------------------------|----------------|
| Hosting | Docker en VPS Hostinger | Vercel static |
| Steps | 5 | 7 (5 free + gate + premium) |
| Campos | 33 | 33-85 (depende del plan) |
| Webhook | `/formulario-clinico` | `/cuestionario-v2` |
| Informe destino | nm_patients (manual) | nm_informes_paciente (auto) |
| Tabla destino | clinica_respuestas_pacientes | misma + columnas premium |
| agent_id | `nextbot` | `cuestionario_v2` |

El v1 sigue activo y NO se ve afectado por este deploy.

## Status

✅ HITO 4.H · Cuestionario v2 desplegado.

Pendiente HITO 4.G para el workflow n8n `cuestionario-v2`.
