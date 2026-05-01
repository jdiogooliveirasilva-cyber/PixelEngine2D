# PixelEngine 2D — v6.2.3

Versão de polimento. Pega 4 melhorias específicas do branch experimental
v6.2.17 e traz pra cima da base estável v6.2.2 (engine offline, sem
Network), sem trazer nada mais junto.

## Novidades

### 1. Projeto sempre em tela cheia
Ao apertar **▶ Jogar** o overlay de runtime entra em fullscreen
automaticamente (pede `requestFullscreen` do elemento `#runtime`).
Funciona em Chrome/Edge desktop e Android. iOS Safari ignora a chamada
e mantém só a tela cheia visual do overlay (limitação do próprio
browser).

Sair com **■ Parar** ou **Esc** chama `exitFullscreen()` e desbloqueia
a orientação.

No HTML exportado, um overlay "▶ Toque para começar" recebe o primeiro
gesto do usuário e dispara fullscreen + landscape juntos. Isso é
necessário porque os browsers só permitem essas APIs dentro de um
gesto explícito.

### 2. Projeto sempre em paisagem
Depois do fullscreen, chama `screen.orientation.lock("landscape")` —
no Android trava de verdade na horizontal. Desktop ignora silencioso.
Adiciona também os meta-tags `screen-orientation` / `x5-orientation`
no HTML exportado pra browsers chineses (UC, X5/QQ).

### 3. Nova ação **Definir tipo de rotação**
Disponível em `OBJETO — VISUAL` no menu de ações. Estilo GDevelop:

- **Todas** — rotação livre 360° (default; comportamento de antes).
- **Apenas horizontal** — ignora ângulo, só vira esquerda/direita
  (espelho via flipX).
- **Sem rotação** — trava o desenho em 0° mesmo se `obj.angle` mudar.
- **Limitada** — mantém o sprite "em pé". Quando o ângulo passa de
  ±90° o sprite é espelhado em vez de virar de cabeça pra baixo.

Importante: a ação NÃO mexe em `obj.angle` nem em `obj.flipX` — ela
só configura como o renderer DESENHA o sprite. Mira (`aim_at`),
`Definir ângulo`, `Rotacionar (Δ°)` e física continuam funcionando
normalmente em cima do ângulo raw. O ajuste vai pra duas propriedades
de display (`_dispAngle` e `_dispFlipX`), aplicadas a cada frame antes
do render.

### 4. Câmera aceita expressões
Os campos numéricos de **cam_zoom**, **cam_zoom_smooth**, **cam_zoom_add**
e **cam_shake** agora passam por `evalExpr()`, então aceitam coisas
como `Player.X()/100` ou `(Hp.health()+1)/2`.

Três ações novas no grupo CÂMERA:

- **Definir X da câmera (expressão)** — `Camera X = (Player.X()+Player2.X())/2`
- **Definir Y da câmera (expressão)** — mesma ideia no eixo Y
- **Definir posição da câmera (X e Y)** — define os dois juntos,
  campo vazio mantém o eixo. Limpa qualquer "Seguir objeto" ativo
  pra não ser sobrescrito no próximo frame.

## Correções de bugs

### A. Editor de pontos não rolava em imagens grandes
Sprites altos (acima de ~360 px) ficavam com a metade de baixo cortada
no editor de pontos — não dava pra ver nem clicar nos pontos da parte
inferior. Agora:

- O wrap do canvas ganha **scroll vertical** (`overflow:auto`) com
  altura máxima de 55% da tela, então qualquer imagem cabe.
- Ao abrir, o zoom é **auto-ajustado** pra que a imagem inteira caiba
  no espaço disponível (cálculo feito uma vez com base no tamanho real
  do contêiner).
- O `touch-action` do wrap permite **arrastar a tela com o dedo** sem
  bloquear o gesto de adicionar/mover ponto no canvas.

### B. Bordas pretas em tela cheia
Antes, no fullscreen, ficavam duas faixas pretas nas laterais (ou em
cima/embaixo) porque o canvas mantinha o aspect ratio da cena via
letterbox CSS. Agora o `resize()` usa modo **fill** estilo GDevelop:

- O canvas ocupa 100% da tela.
- A altura da cena (`scene.height`) vira a referência de escala — os
  sprites continuam do tamanho que você desenhou.
- O `viewWidth` da câmera é recalculado pra o aspecto real do
  dispositivo, então a câmera passa a enxergar **mais ou menos largura
  de mundo** conforme a tela. Sem distorção, sem corte vertical, sem
  faixa preta.

Listeners adicionais em `fullscreenchange` e `screen.orientation`
garantem que o ajuste seja refeito ao entrar/sair de fullscreen ou
girar o aparelho.

### C. Câmera começava muito baixa
Quando a câmera tinha "Seguir objeto" ativo, ela nascia no centro da
cena (`scene.width/2, scene.height/2`) e o smoothing 0.1 levava vários
frames pra alcançar o player — efeito visual: a câmera aparecia
deslocada pra baixo e o chão sumia nos primeiros segundos de jogo.

Agora, no início da cena (e a cada troca de cena), a câmera dá um
**snap instantâneo** na posição do alvo de follow. O smoothing
continua valendo a partir do frame seguinte — só o frame 0 é colado
pra evitar o "drift" inicial.

## O que NÃO mudou

- Engine continua 100% offline (Network removido na v6.2.2 — sem volta).
- Nenhuma outra ação, condição ou expressão foi alterada.
- Projetos salvos da v6.2.2 abrem direto na v6.2.3 sem migração.
