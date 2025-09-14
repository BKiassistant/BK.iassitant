bk-iassistant/


const supabaseAdmin = createClient(process.env.NEXT_PUBLIC_SUPABASE_URL, process.env.SUPABASE_SERVICE_ROLE_KEY)


export default async function handler(req, res) {
if (req.method !== 'POST') return res.status(405).end()
const token = req.headers.authorization?.split(' ')[1]
if (!token) return res.status(401).json({ error: 'No autorizado' })


// 1) Verificar usuario con token Supabase
const { data: { user }, error } = await supabaseAdmin.auth.getUser(token)
if (error || !user) return res.status(401).json({ error: 'Usuario no encontrado' })


// 2) (Opcional) comprobar subscripción en la tabla 'subscriptions'
const { data: subs } = await supabaseAdmin.from('subscriptions').select('*').eq('user_email', user.email).limit(1)
if (!subs || subs.length === 0) return res.status(402).json({ error: 'Debes tener una suscripción activa' })


const { prompt } = req.body
if (!prompt) return res.status(400).json({ error: 'Falta prompt' })


// 3) Llamar a OpenAI
try {
const openaiRes = await fetch('https://api.openai.com/v1/chat/completions', {
method: 'POST',
headers: {
'Content-Type': 'application/json',
'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`
},
body: JSON.stringify({
model: 'gpt-4o-mini',
messages: [{ role: 'user', content: prompt }],
max_tokens: 800
})
})


const data = await openaiRes.json()
const reply = data?.choices?.[0]?.message?.content || data?.error || 'Sin respuesta'
res.status(200).json({ reply })
} catch (err) {
console.error('OpenAI error', err)
res.status(500).json({ error: 'Error en la llamada a OpenAI' })
}
}
```


---


## `styles/globals.css`
```css
@tailwind base;
@tailwind components;
@tailwind utilities;


:root{
--brand: #0a2540;
--accent: #1f6feb;
}


body { background: #f8fafc; }


/* Utilities for colors used inline in components */
.bg-brand { background-color: var(--brand); }
.text-brand { color: var(--brand); }
.bg-accent { background-color: var(--accent); }
```
