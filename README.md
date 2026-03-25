# ✈️ VoaFácil Ultra — README Completo

## 🚀 9 Diferenciais que ninguém tem no Brasil

| # | Diferencial | Status | Receita |
|---|-------------|--------|---------|
| 1 | 🤖 IA de Previsão de Preço | Construir com histórico Amadeus | Retenção |
| 2 | 🗺️ Modo Sonho — Mapa de Destinos | Mapbox GL JS + Amadeus | Viral / Aquisição |
| 3 | ⚡ Clube de Erros de Tarifa | Monitor de preços + threshold | Assinaturas Pro |
| 4 | 📅 Calendário de Preços | Amadeus `/v1/shopping/flight-dates` | Conversão |
| 5 | 💎 Combinador de Milhas | APIs LATAM Pass + Smiles | Diferenciação |
| 6 | 👫 Split com Amigos | Stripe Payment Links | Viral / Aquisição |
| 7 | 💻 Modo Nômade Digital | Algoritmo TSP + Amadeus | Nicho premium |
| 8 | 🔒 Seguro de Preço | Stripe + lógica de hedge | Margem pura |
| 9 | 🚨 Plantão Emergencial 24/7 | WhatsApp + operadores | B2B recorrente |

---

## 💰 Modelo de Receita Completo

### Fluxo 1 — Assinaturas SaaS
| Plano | Preço | Usuários alvo |
|-------|-------|---------------|
| Explorer | Grátis | Validação de mercado |
| Voyager Pro | R$29/mês | Viajantes frequentes |
| Nômade | R$59/mês | Nômades digitais |
| Elite Agência | R$299/mês | Agências + Plantão |

### Fluxo 2 — Plantão Emergencial B2B
| Plano | Preço | Margem |
|-------|-------|--------|
| Essencial | R$99/mês | ~70% |
| Profissional | R$299/mês | ~75% |
| Enterprise | R$999/mês | ~80% |

**340 agências × R$299 médio = R$101.660/mês** só no Plantão.

### Fluxo 3 — Seguro de Preço
- R$15–25 por travamento
- Margem média: ~60% (risco coberto pelo histórico de preços)
- Meta: 2.000 seguros/mês = **R$30.000–50.000/mês**

### Fluxo 4 — Comissões de Afiliado
- Travelpayouts: ~R$15–40 por passagem vendida
- LATAM/GOL direto: 2–3% do ticket
- Meta: 1.500 vendas/mês × R$800 médio × 2.5% = **R$30.000/mês**

### Fluxo 5 — Clube de Erros de Tarifa (Upsell)
- Alertas antecipados exclusivos para assinantes Pro+
- Drives upgrade de Explorer → Voyager

### Resumo de MRR potencial em 12 meses
```
Assinaturas:        R$ 72.800
Plantão B2B:        R$ 42.100
Seguro de preço:    R$  8.700
Comissões:          R$ 18.400
─────────────────────────────
TOTAL MRR:          R$142.000/mês
ARR Projetado:      R$1.704.000
```

---

## 🚨 Implementação do Plantão Emergencial

### Estrutura da equipe
```
Turno manhã    (06h–14h): 3 agentes
Turno tarde    (14h–22h): 4 agentes (pico)
Turno noite    (22h–06h): 2 agentes + 1 supervisor
```

**Custo estimado por agente:** R$2.500–4.000/mês
**Equipe de 9 agentes:** R$27.000–36.000/mês
**Receita com 100 agências:** ~R$30.000/mês → break-even
**Com 340 agências:** R$101.000 → **margem de R$65.000/mês**

### Tipos de incidente e SLA
| Tipo | Prioridade | SLA |
|------|-----------|-----|
| Cancelamento de voo | P1 | 8 min |
| Conexão perdida | P1 | 8 min |
| Emergência médica | P1 | 8 min |
| Overbooking hotel | P2 | 20 min |
| Bagagem extraviada | P2 | 20 min |
| Problema documentação | P2 | 20 min |
| Disputa de reembolso | P3 | 60 min |

### Ferramentas que os agentes usam
1. **Amadeus Travel APIs** — reemissão de bilhetes
2. **Booking.com Extranet** — realocação de hotel
3. **Contatos diretos das aéreas** — tabela no banco
4. **WhatsApp Business API** — comunicação com pax
5. **Seguradora parceira API** — acionamento de seguros
6. **Google Translate** — suporte multilíngue

### Como vender para agências
- Cold outreach: ABAV, ABRACORP, Sindetur
- Demo: "Seu cliente ficou preso? Liga pra gente. Resolvemos em 8 minutos."
- Proposta de valor: 1 caso resolvido = cliente fiel pela vida toda
- ROI fácil: 1 cliente salvo vale R$10.000+ em reservas futuras

---

## 🤖 IA de Previsão de Preço — Implementação

```typescript
// lib/flights/ai-predictor.ts

// Modelo simples de regressão linear com dados do Amadeus
// Para produção: use Prophet (Python) ou AWS Forecast

export function predictPriceDirection(priceHistory: number[]): {
  direction: 'up' | 'down' | 'stable'
  confidence: number
  expectedChange: number
} {
  const n = priceHistory.length
  if (n < 7) return { direction: 'stable', confidence: 0.5, expectedChange: 0 }

  // Regressão linear simples
  const x = priceHistory.map((_, i) => i)
  const y = priceHistory
  const xMean = x.reduce((a,b) => a+b,0) / n
  const yMean = y.reduce((a,b) => a+b,0) / n
  const slope = x.reduce((s,xi,i) => s + (xi-xMean)*(y[i]-yMean), 0)
             / x.reduce((s,xi) => s + Math.pow(xi-xMean,2), 0)

  // Correlação para calcular confiança
  const r = slope * (n * x.reduce((s,xi,i) => s + xi*y[i],0) - x.reduce((a,b)=>a+b,0)*y.reduce((a,b)=>a+b,0))
  const confidence = Math.min(0.95, Math.abs(r / 1000) + 0.5)

  const projectedChange = slope * 7  // projeção de 7 dias

  return {
    direction: slope > 5 ? 'up' : slope < -5 ? 'down' : 'stable',
    confidence: Math.round(confidence * 100) / 100,
    expectedChange: Math.round(projectedChange),
  }
}
```

---

## 🗺️ Modo Sonho — Implementação

```typescript
// app/api/dream-mode/route.ts

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const origin = searchParams.get('origin') ?? 'GRU'
  const budget = parseFloat(searchParams.get('budget') ?? '2000')
  const month  = searchParams.get('month') ?? '2025-04'

  // Busca destinos populares no mês
  const destinations = [
    'MIA','LIS','CDG','FCO','MAD','AMS','LHR','EZE',
    'SCL','BOG','MEX','CUN','BCN','MXP','GRU','SSA',
  ]

  const results = await Promise.allSettled(
    destinations.map(async (dest) => {
      const res = await fetch(
        `https://api.amadeus.com/v1/shopping/flight-destinations?origin=${origin}&maxPrice=${budget}`,
        { headers: { Authorization: `Bearer ${await getAmadeusToken()}` } }
      )
      const data = await res.json()
      return data.data?.find((d: any) => d.destination === dest)
    })
  )

  const affordable = results
    .filter(r => r.status === 'fulfilled' && r.value)
    .map((r: any) => r.value)
    .filter(d => d.price.total <= budget)
    .sort((a, b) => a.price.total - b.price.total)

  return Response.json({ destinations: affordable, origin, budget })
}
```

---

## ⚡ Clube de Erros de Tarifa — Implementação

```typescript
// lib/flights/fare-error-detector.ts

const ERROR_THRESHOLD = 0.40  // preço < 40% da média histórica = erro

export async function detectFareErrors(origin: string, destination: string) {
  const [current, history] = await Promise.all([
    searchAllSources({ origin, destination, departureDate: nextMonth(), adults: 1 }),
    getPriceHistory(origin, destination, 90),  // 90 dias de histórico
  ])

  if (!current.length || !history.length) return []

  const avgPrice = history.reduce((s, p) => s + p, 0) / history.length
  const threshold = avgPrice * ERROR_THRESHOLD

  return current
    .filter(f => f.price < threshold)
    .map(f => ({
      ...f,
      normalPrice: Math.round(avgPrice),
      discount: Math.round((1 - f.price / avgPrice) * 100),
      isError: true,
      expiresAt: new Date(Date.now() + 2 * 60 * 60 * 1000),  // estimativa: 2h
    }))
}
```

---

## 🔒 Seguro de Preço — Implementação

```typescript
// app/api/price-lock/route.ts

export async function POST(request: Request) {
  const { flightId, price, userId } = await request.json()

  // Cobra R$20 pelo seguro via Stripe
  const paymentIntent = await stripe.paymentIntents.create({
    amount: 2000,  // R$20,00
    currency: 'brl',
    metadata: { flightId, lockedPrice: price, userId },
  })

  // Salva no banco com expiração de 48h
  await supabase.from('price_locks').insert({
    user_id: userId,
    flight_id: flightId,
    locked_price: price,
    expires_at: new Date(Date.now() + 48 * 60 * 60 * 1000),
    stripe_payment_intent: paymentIntent.id,
  })

  return Response.json({ clientSecret: paymentIntent.client_secret })
}
```

---

## 📦 Setup Completo

```bash
# 1. Clone e instale
git clone https://github.com/seu-usuario/voafacil-ultra
cd voafacil-ultra
npm install

# 2. Configure variáveis
cp .env.example .env.local
# Preencha: Supabase, Amadeus, Stripe, Z-API, Resend

# 3. Rode o schema do banco
# Supabase SQL Editor → rode supabase/schema.sql
# Depois rode supabase/emergency-schema.sql

# 4. Rode localmente
npm run dev

# 5. Deploy
vercel deploy --prod
```

---

## 📅 Roadmap Priorizado

### Semana 1–2: Core MVP
- [ ] Busca Amadeus funcionando
- [ ] Auth Supabase
- [ ] Alertas WhatsApp básicos
- [ ] Landing page ao vivo

### Semana 3–4: Diferenciação
- [ ] Calendário de preços
- [ ] Detector de erros de tarifa
- [ ] Seguro de preço (Stripe)

### Mês 2: Plantão + Escala
- [ ] Painel do plantão (este arquivo)
- [ ] Onboarding primeiras 10 agências
- [ ] IA de previsão de preço
- [ ] Modo Sonho com mapa

### Mês 3+: Monetização máxima
- [ ] Combinador de milhas
- [ ] Split com amigos
- [ ] Modo Nômade Digital
- [ ] App mobile
- [ ] Programa de afiliados

---

## 🏆 Por que isso vence a concorrência

| Feature | VoaFácil Ultra | Google Flights | Decolar | MaxMilhas |
|---------|---------------|----------------|---------|-----------|
| IA previsão de preço | ✅ | ✅ (EUA) | ❌ | ❌ |
| Modo Sonho mapa | ✅ | ✅ | ❌ | ❌ |
| Erros de tarifa | ✅ | ❌ | ❌ | ❌ |
| Calendário de preços | ✅ | ✅ | ❌ | ❌ |
| Combinador de milhas | ✅ | ❌ | ❌ | Parcial |
| Split com amigos | ✅ | ❌ | ❌ | ❌ |
| Modo Nômade | ✅ | ❌ | ❌ | ❌ |
| Seguro de preço | ✅ | ❌ | ❌ | ❌ |
| Plantão 24/7 | ✅ | ❌ | ❌ | ❌ |
| WhatsApp nativo BR | ✅ | ❌ | ❌ | ❌ |
| Foco 100% no BR | ✅ | ❌ | Parcial | ✅ |
