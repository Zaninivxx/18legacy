# 18 Legacy — site (prévia v1)

É um site estático de uma página: todo o HTML, CSS e JavaScript está no `index.html`.
Não tem build nem `npm install`. As bibliotecas carregam por CDN.

| Biblioteca | Versão | Uso |
|---|---|---|
| three.js | r128 | ∞ de partículas em 3D |
| GSAP + ScrollTrigger | 3.12.5 | Animações e cenas fixadas na rolagem |
| Lenis | 1.1.13 | Rolagem suave (só com mouse ou trackpad) |

As fontes Cinzel e Jost vêm do Google Fonts.
O som é gerado pela Web Audio API, sem arquivos de áudio.

---

## 1. Publicar

### GitHub
```bash
git init
git add .
git commit -m "Prévia v1 do site da 18 Legacy"
git branch -M main
git remote add origin git@github.com:SEU-USUARIO/18-legacy-site.git
git push -u origin main
```

### Vercel
1. Vá em Add New → Project e importe o repositório.
2. Em **Framework Preset**, escolha **Other**.
3. Deixe o **Build Command** vazio.
4. Em **Output Directory**, use `.`.

O `vercel.json` já traz cabeçalhos de segurança e cache. Ele também libera o giroscópio para o próprio site (`Permissions-Policy`), o que é necessário para a inclinação no celular.

### Domínio (quando for a hora)
Vá em Settings → Domains, adicione `18legacy.com.br` e configure o DNS conforme a Vercel indicar.

### Rodar localmente
```bash
npx serve .
# ou
python3 -m http.server 5173
```

> Teste no celular pela rede local (`http://IP-DO-PC:5173`) ou, melhor, pelo link de preview da Vercel. O giroscópio só funciona em **HTTPS**.

---

## 2. Adaptação automática ao aparelho

Ao abrir, o site mede o aparelho e escolhe um **nível de efeitos**. O nível fica gravado em `<html data-fx="...">`.

| Nível | Quando | O que muda |
|---|---|---|
| `high` | Computador com 8+ núcleos e 8+ GB, com mouse | 8.200 partículas, nitidez até 1,8×, grão de filme animado, rolagem suave |
| `mid` | Celular ou tablet moderno, ou máquina de 4 núcleos ou 4 GB | 3.800 partículas um pouco maiores, nitidez até 1,5×, grão parado, rolagem nativa do toque |
| `low` | Economia de dados, rede 2G, até 2 GB, GPU fraca ou por software, ou "reduzir movimento" | 1.700 partículas, nitidez 1×, grão parado |
| `off` | Sem WebGL, ou a GPU caiu no meio (context lost) | O ∞ vira um SVG estático em ouro; todo o resto funciona |

Além da detecção inicial, há dois mecanismos que agem durante o uso:

- **Medidor de fluidez:** nos primeiros 90 quadros com o ∞ na tela, o site mede o FPS real.
  - Abaixo de ~25 fps: corta as partículas para 45% e reduz a nitidez para 1×.
  - Abaixo de ~38 fps: corta as partículas para 70%.
  - Isso pega aparelhos que "parecem" bons mas travam, como iPhones em Modo de Pouca Energia.
- **Economia fora do topo:** quando o ∞ some (depois do topo) ou a aba fica em segundo plano, o 3D para de desenhar.

**Forçar um nível para testar:** `?fx=high`, `?fx=mid`, `?fx=low` ou `?fx=off` no fim da URL.

**Movimento reduzido:** se o aparelho tem "reduzir movimento" ligado, as cenas não ficam fixadas, as animações de rolagem não rodam e todo o conteúdo aparece estático e completo.

---

## 3. Efeitos no celular

No toque não existe hover, então cada efeito tem um equivalente:

| No computador | No celular |
|---|---|
| Passar o mouse no ∞ desfaz as partículas em poeira | **Tocar no ∞** espalha a poeira, e **rolar a página** sopra a poeira de leve |
| O ∞ acompanha o mouse | O ∞ acompanha a **inclinação do aparelho** (giroscópio) |
| Os visuais de "Como funciona" inclinam com o mouse | Os visuais inclinam com o aparelho |
| Hover no manifesto, nas fases e na Regra | O item que **passa pelo centro da tela acende** sozinho, com a nota de som se o som estiver ligado |

**Sobre o giroscópio no iPhone:** o iOS exige permissão. O pedido aparece no primeiro toque no ∞ ou dentro de "Como funciona". No Android funciona direto.

---

## 4. Layout por tamanho de tela

| Tela | Comportamento |
|---|---|
| Celular em pé | ∞ no terço superior, título embaixo. "Como funciona" fica fixado, com o visual em cima e o texto embaixo. Fases em lista vertical. Estante com 2 fileiras de 9 volumes, em meia altura |
| Celular deitado ou tela baixa (menos de 600px de altura) | ∞ à direita do título. "Como funciona" e as fases **não** ficam fixados, porque não caberiam: tudo rola normalmente |
| Tablet em pé | Igual ao celular, com mais respiro |
| Tablet deitado, notebook e desktop | ∞ à direita do título. "Como funciona" fixado com o texto à esquerda e o visual à direita. Fases em rolagem horizontal |

Detalhes que valem para todas as telas:
- O cabeçalho ganha fundo sólido depois do topo, para ficar legível também sobre o bloco Marfim do Livro.
- O botão de som fica no cabeçalho, sem nada flutuando sobre o texto.
- Os insets de notch e barra inferior (`safe-area`) são respeitados.
- Os campos do formulário usam 16px, o que evita o zoom automático do iPhone.

---

## 5. O que foi testado

Os testes rodaram em Chromium automatizado, com as fontes reais (Cinzel e Jost). Em cada aparelho, a página foi percorrida de ponta a ponta, checando três coisas: transbordo horizontal, texto cortado e erros de console.

| Aparelho | Tela | Resultado |
|---|---|---|
| iPhone SE (1ª geração) | 320×568 | ok |
| Galaxy A | 360×740 | ok |
| iPhone SE (2ª/3ª) | 375×667 | ok |
| iPhone 13 | 390×844 | ok |
| iPhone Pro Max | 430×932 | ok |
| Celular deitado | 844×390 | ok |
| iPad | 768×1024 | ok |
| iPad deitado | 1024×768 | ok |
| Notebook | 1366×768 | ok |
| Desktop | 1920×1080 | ok |
| Modo `?fx=off` | — | ok |
| Modo `?fx=high` forçado | — | ok; o medidor de fluidez rebaixou sozinho no renderizador de software |

**Falta testar em aparelho físico** (o teste automatizado não cobre):
- Safari real no iPhone: giroscópio, som e rolagem com a barra de endereço sumindo.
- Um Android de entrada.

Na prática: abra o link da Vercel no seu celular e no do Igor, e confira no console qual nível foi escolhido (`document.documentElement.dataset.fx`).

---

## 6. Limitações conhecidas

- **Som no iPhone:** com a chave de silencioso ligada, o iOS pode silenciar o som da página. Isso é comportamento do sistema.
- **Lista de espera:** o formulário **não envia** dados. Ele valida os campos e mostra um aviso de prévia.
  - Para ligar: uma tabela no Supabase (já previsto para o app) e uma rota de envio.
  - Antes disso: termo de consentimento e política de privacidade (LGPD).
- **Logo:** entra a oficial no lugar do texto "18 LEGACY" e do `favicon.svg` provisório.
- **Imagem de compartilhamento (og:image, 1200×630):** falta criar, para o link ter prévia no WhatsApp.
- **Textos que aguardam aprovação do Igor:**
  - primeiro passo de "Como funciona";
  - descrições das fases;
  - título e sequência do Livro do Legado;
  - frase de abertura do manifesto;
  - lâmina do banco no manifesto (falta o texto aprovado).
