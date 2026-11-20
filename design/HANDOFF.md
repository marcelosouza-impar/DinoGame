# DinoGame · Handoff da direção visual "Bubble Day"

Este pacote leva para o código a nova direção visual criada no canvas de design (link no fim). O objetivo é trocar o visual escuro neon e a fonte pixel minúscula por um estilo cartoon infantil: céu claro, contorno escuro grosso, sombra sólida, cantos arredondados e movimento com mola.

## Como usar este pacote

| Arquivo | Para que serve |
|---|---|
| `tokens.css` | Variáveis de cor, fonte, forma e movimento, mais as classes base de botões, pílulas, cartões e teclas. Fonte da verdade do estilo. |
| `dino/dino.svg` | Novo desenho do Dino em peças separadas (rabo, espinhos, pernas, tronco, braço, cabeça de perfil, cabeça frontal). |
| `dino/dino-animations.css` | Animações em CSS por modo (`m-idle`, `m-jump`, `m-duck`, `m-wait`). Servem para telas HTML (menu, seleção). |
| `dino/poses.json` | As mesmas animações em números (poses-chave, tempos, escalas, giros) para portar ao desenho em canvas. |
| `reference/*.dc.html` | Telas de referência. Formato do editor de design, mas o HTML e os estilos inline podem ser lidos direto como especificação. Não abrir no navegador: dependem de um runtime. |

Telas de referência: `Main` (menu), `Mapa` (mapa das fases), `Hud` (HUD da fase), `JogoLivre` (Jogo Livre 3D), `Editor` (criar fase), `Kit` (componentes), `Dino` (ficha de animação).

## Regras para a implementação

1. O jogo continua em arquivo único (`index.html`). Inline o conteúdo de `tokens.css` dentro do `<style>` existente; não crie dependências novas além das fontes do Google.
2. Uma fase por vez, um commit por fase. Rodar o jogo e conferir antes de seguir.
3. Não mudar regras de jogo, física, saves ou chaves de `localStorage`. Só visual e UI, salvo quando a fase disser o contrário.
4. Texto mínimo de 12 px no canvas de 800×450. Nunca mais Press Start 2P em texto corrido (pode ficar em números do placar, se quiser).
5. Alvos de toque com no mínimo 44 px. Um botão primário (amarelo) por tela.
6. Ícones: SVG inline com traço (stroke 2.6 a 3, cor `--ink`). Remover emoji da interface.
7. Contraste: texto sempre em `--ink` sobre as cores claras. Não usar texto branco sobre coral, amarelo ou verde.

## Fases (ordem do mais barato ao mais caro)

### Fase 1 · Fontes, cores e botões (CSS)
- Inline `tokens.css`. Trocar `font-family` do body para `--font-ui`; títulos `h1/h2` para `--font-display`.
- Reescrever `.btn` como `.btn-primary`, `.btn.sec` como `.btn-secondary`, `.hbtn` como `.btn-icon`, `.ocard` como `.card` com fundo `--cream`.
- Remover os estilos inline dos botões gerados em `renderMainMenu`, `showCharSelect`, `renderWorldMap` e afins (gradientes roxo, azul e lilás). Hierarquia por tamanho, não por cor.
- Corrigir todos os `font-size` abaixo de 12 px (há casos de 3,8 a 8,5 px em `.skin-card`, `.wardrobe-card`, `.album-node`, `.credits`, `.sub`, `.khint`).
- `#overlay`: trocar fundo escuro com blur por `--sky` (ou cena com morros). `#gameContainer`: remover o brilho neon.
- **Pronto quando:** nenhuma tela de menu tem texto menor que 12 px, nenhum emoji em botão, e só existe um botão amarelo por tela.

### Fase 2 · Menu principal e mapa
- Menu conforme `reference/Main.dc.html`: título "Dino / Game" inclinado com contorno, botão "Jogar" primário, "Jogo Livre" secundário, "Criar Fase" bloqueado e visível com o custo (700 moedas), atalhos como ícones quadrados arredondados com rótulo, pílulas de nome e moedas no topo, engrenagem que junta idioma e som.
- Mapa conforme `reference/Mapa.dc.html`: trilha com nós circulares (atual em amarelo com balão "Você está aqui", bloqueados em branco com cadeado), doca inferior com Loja, Guarda-roupa, Álbum e Base.
- **Pronto quando:** todas as ações que existiam continuam acessíveis.

### Fase 3 · Dino novo
- Telas HTML (menu, seleção de personagem): usar `dino/dino.svg` + `dino-animations.css`, modo `m-idle`.
- Dentro do jogo (canvas): reescrever `drawDinoOn(c,x,y,frame,ducking,s)` desenhando as mesmas peças com `Path2D` (os `d` do SVG funcionam direto em `new Path2D(d)`), na ordem: sombra, rabo, espinhos, perna de trás, tronco, barriga, perna da frente, braço, cabeça.
- Animação: aplicar as poses de `poses.json` com `ctx.translate/scale/rotate` por peça. Escala com origem nos pés.
  - **Pulo:** usar a curva de `jump.keys` sincronizada com a velocidade vertical real (subindo = impulso/topo, descendo = queda, ao tocar o chão = aterrissar por ~80 ms). Não trocar a física.
  - **Abaixar:** pose `duck` enquanto `ducking` for verdadeiro, com contraescala na cabeça.
  - **Esperando:** se não houver input por 4 s em estado de jogo, trocar para cabeça frontal com o "pop" e o saltinho; qualquer tecla ou toque volta para `idle` e zera o timer.
- **Pronto quando:** o Dino lê bem em tamanho real, as três sequências funcionam e o hitbox não mudou.

### Fase 4 · HUD da fase
- Conforme `reference/Hud.dc.html`: vidas em corações dentro de pílula, moedas em pílula, barra de progresso da fase com bandeira, botões redondos de música e pausa, controles mobile redondos (Pular amarelo grande, Fogo laranja).

### Fase 5 · Jogo Livre (raycaster)
- Conforme `reference/JogoLivre.dc.html`. Céu de dia, chão creme, estrada `--road` com faixa central amarela, paredes com as cores da paleta (face iluminada e face em sombra).
- Janelas: textura procedural pela coordenada de impacto na parede (sem imagens).
- Contorno: faixa vertical `--ink` quando muda o tipo de parede ou quando a distância dá um salto entre colunas vizinhas; linha `--ink` no topo e na base das paredes.
- HUD: pílula de moedas, chip do bairro, teclas desenhadas (W A S D, C), minimapa redondo.
- **Atenção:** topo arredondado de prédio não sai em raycaster; usar faixa mais escura no alto da parede.

### Fase 6 · Editor de fases
- Conforme `reference/Editor.dc.html`. Primeiro só visual: blocos novos, gaveta grande com números, botão "Testar" no lugar do ENTER, alternar Colocar/Apagar, desfazer em botão, barra de posição na fase.
- Depois, como funcionalidade nova (combinar antes): refazer e abas Itens e Inimigos com posicionamento livre. Isso muda o formato salvo de `customPhaseConfig`; manter compatibilidade com fases antigas.

## Link do canvas de design
https://claude.ai/artifact/2VHz9HRpKS1qqjT3WuTfeW
