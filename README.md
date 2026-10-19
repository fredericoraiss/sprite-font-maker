# Sprite Font Maker

Transforma uma **sprite sheet de glifos** (uma imagem com todas as letras desenhadas) em uma **BitmapFont para Godot 3.x**: um arquivo `.fnt` (formato BMFont) e um `.png` (atlas).

O foco do programa é resolver dois problemas comuns de spritefonts:

- **Distância grande entre as letras**: cada glifo é recortado pelos pixels reais e os pares de letras são encaixados por kerning automático.
- **Acentuação**: os glifos acentuados que já existem na sheet (Á, Ã, Ç, Ñ…) viram caracteres Unicode normais, e as minúsculas reaproveitam os glifos maiúsculos.

É uma página web local (um único `index.html`, sem instalação, sem internet, sem servidor). Nada é enviado para fora do seu computador.

---

## Como usar (passo a passo)

1. Abra o `index.html` no navegador (duplo clique).
2. Arraste a sprite sheet para a área tracejada, ou clique nela para escolher o arquivo (PNG ou WebP, de preferência com fundo transparente).
3. O programa detecta a grade e os glifos sozinho. Confira na aba **Sheet e glifos**: cada glifo ganha uma caixa verde com o caractere atribuído.
4. Se a ordem dos caracteres estiver diferente da sua sheet, ajuste o campo **Caracteres** (seção 3) ou clique no glifo errado e corrija.
5. Ajuste o espaçamento (seção 4) olhando a aba **Prévia do texto**.
6. Em **Exportar** (seção 5), dê um nome à fonte e clique em **Baixar .zip**.
7. Extraia o `.zip` dentro do projeto Godot 3.x, **com o `.fnt` e o `.png` na mesma pasta**, e use o `.fnt` como *Font* nos nós de texto (o Godot o carrega como `BitmapFont`).

---

## Requisitos da sprite sheet

- Os glifos ficam dispostos em **grade** (linhas e colunas com passo regular), em **ordem de leitura** (esquerda para direita, de cima para baixo).
- Fundo transparente é o ideal. Se a imagem tiver fundo sólido (os 4 cantos opacos e da mesma cor), o programa remove essa cor automaticamente.
- Os glifos **podem ultrapassar o limite da célula** (G, Q e Ç com a parte de baixo, por exemplo) e podem estar **colados** ao vizinho (como Æ e É em algumas sheets): o programa separa os dois.
- O alinhamento vertical é calculado pela linha de base de cada linha da sheet, então a deriva de alguns pixels entre linhas não atrapalha.

---

## A tela, ponto a ponto

### 1 · Sheet

| Controle | O que faz |
|---|---|
| Área de arrastar | Carrega a imagem. Também funciona soltar o arquivo em qualquer lugar da página. |
| **Limiar de alpha** | Pixels com transparência abaixo desse valor (0–255) são tratados como vazio. Aumente se houver "sujeira" fraca ao redor dos glifos; diminua se bordas suaves estiverem sendo cortadas. Padrão: 8. |

### 2 · Grade da sheet

A grade é detectada automaticamente ao carregar a imagem.

| Controle | O que faz |
|---|---|
| **Passo X / Y** | Largura e altura de cada célula da grade, em pixels. É usado para decidir a que célula (linha e coluna) cada glifo pertence. |
| **Origem X / Y** | Posição da divisa da primeira célula. |
| **Redetectar grade** | Refaz a detecção automática (útil depois de mexer nos números à mão). |

Só mexa nesses números se os glifos aparecerem agrupados na célula errada. Valores aproximados funcionam, porque os glifos são atribuídos pelo centro e só a divisa entre as células precisa cair no espaço vazio entre eles.

### 3 · Caracteres (ordem de leitura)

Diz ao programa qual caractere é cada glifo da sheet.

| Controle | O que faz |
|---|---|
| **Modelo** | Ordens prontas. Ao carregar a imagem, o programa escolhe o modelo cujo tamanho bate com a quantidade de glifos. Opções: *Latim PT/ES/FR (96)*, *A–Z + 0–9 (36)*, *ASCII 33–126 (94)* e *Personalizado*. |
| **Caixa de texto** | A ordem dos caracteres, na ordem de leitura da sheet. Espaços e quebras de linha são ignorados. Editar o texto troca o modelo para *Personalizado*. |
| **Minúsculas usam os glifos maiúsculos** | Se a sheet só tem maiúsculas, `a` e `á` passam a usar os glifos de `A` e `Á`. Não sobrescreve minúsculas que existam de verdade na sheet. |
| **Aliases** | Aproveita o glifo de `'` para ’ ‘, de `"` para “ ”, e de `-` para – —. |
| **Ignorar células vazias ao contar** | Por padrão, a posição de cada glifo conta pela célula (uma célula vazia pula um caractere). Ligando essa opção, só os glifos existentes são contados em sequência. |

O programa mostra quantos glifos detectou e quantos caracteres tem a ordem, e avisa se houver mais glifos que caracteres.

> **Importante:** o programa **não reconhece a letra olhando o desenho**. Ele usa a ordem do modelo e você confere visualmente na aba *Sheet e glifos*, onde o caractere atribuído aparece sobre cada glifo.

O modelo padrão (96 caracteres) corresponde a esta ordem, 11 por linha:

```
ABCDEFGHIJK
LMNOPQRSTUV
WXYZ0123456
789.,;:?!-_
~#"'&()[]|`
\/@°+=*$£€<
>%ÁÂÀÅÃÄÆÉÊ
ÈËÍÎÌÏÓÔÒÕÖ
ÚÛÙÜÇÑÝ¿
```

### 4 · Espaçamento

| Controle | O que faz |
|---|---|
| **Espaçamento entre letras** | Valor somado ao avanço de todos os glifos. 0 = glifos encostados caixa com caixa. Aceita negativo para sobrepor. |
| **Kerning automático** | Para cada par de letras, mede a distância real entre os contornos (linha por linha) e aproxima os dois até a folga desejada. É o que faz, por exemplo, o `A` entrar embaixo do `T`. Os pares vão para o `.fnt` como linhas `kerning`. |
| **Folga mínima entre letras** | Distância que deve sobrar entre os pixels de duas letras vizinhas depois do kerning. Padrão: 1. Valores menores aproximam mais. |
| **Aperto máximo por par** | Limite de quanto um par pode ser aproximado, em pixels. Evita que letras fiquem sobrepostas demais. Padrão: 40. |
| **Ignorar kerning menor que** | Pares com ajuste menor que esse valor não são gravados. Diminui bastante o tamanho do `.fnt`. Padrão: 2. |
| **Permitir afastar pares sobrepostos** | Se ligado, o kerning também afasta pares que já estão mais apertados que a folga mínima. Desligado, o kerning só aproxima. |
| **Largura do espaço** | Largura do caractere espaço, em pixels. 0 = automático (40% da largura típica de uma maiúscula). |
| **Entrelinha extra** | Pixels somados à altura da linha. A altura base já considera os acentos mais altos e as letras que descem. Aceita negativo. |

### 5 · Exportar

| Controle | O que faz |
|---|---|
| **Nome da fonte** | Nome dos arquivos gerados (`Nome.fnt`, `Nome.png`). Caracteres especiais viram `_`. Se você não editar, usa o nome do arquivo da sheet. |
| **Margem entre glifos no atlas** | Espaço vazio entre glifos no PNG gerado. Evita que o filtro de textura do Godot "sangre" a cor de um glifo no vizinho. Padrão: 2. |
| **Baixar .zip** | Baixa o `.fnt` e o `.png` juntos. |
| **.fnt** / **.png** | Baixa cada arquivo separadamente. |

Os botões só ficam ativos depois que uma sheet é carregada.

---

## As abas

### Sheet e glifos

Mostra a sheet com uma caixa em cada glifo detectado.

- **Zoom**: aumenta ou diminui a visualização.
- **Mostrar caixas** / **Mostrar caracteres**: liga e desliga as caixas e as letras atribuídas.
- Cores: verde = glifo normal, amarelo = selecionado, vermelho = glifo sem caractere ou excluído.
- **Clicar num glifo** abre o painel de ajuste:
  - **Caractere**: troca o caractere só desse glifo.
  - **Avanço extra**: soma largura ao avanço desse glifo.
  - **Deslocar Y**: sobe ou desce o glifo (negativo = sobe).
  - **Excluir da fonte**: tira o glifo do resultado.

### Prévia do texto

Digite um texto e veja como ele ficará, desenhado com o atlas, o kerning e as métricas que serão exportadas. Dá para usar várias linhas.

- **Escala**: 0.5×, 1× ou 2×.
- **Usar kerning**: liga e desliga o kerning na prévia, para comparar.
- **Mostrar caixas dos glifos**: mostra o retângulo de cada glifo, útil para ver o encaixe.
- **Fundo**: escuro, branco, roxo ou xadrez (para conferir transparência).
- Abaixo aparece "Sem glifo para: …" quando o texto tem caracteres que a fonte não possui.
- Embaixo ficam as métricas: `lineHeight`, `base`, largura do espaço, tamanho do atlas, quantidade de caracteres e de pares de kerning.

### Atlas gerado

Mostra o PNG que será exportado.

---

## Como o programa funciona por dentro

1. **Componentes conectados**: acha cada "mancha" de pixels opacos. Acentos costumam estar ligados ao glifo pelo contorno e pela sombra, então já vêm juntos.
2. **Grade**: descobre o passo e a origem da grade a partir dos centros dessas manchas.
3. **Separação de glifos colados**: se uma mancha cobre mais de uma célula (como Æ + É), corta no ponto de menor tinta perto da divisa da grade.
4. **Atribuição**: cada pedaço vai para a célula do seu centro, e o glifo é a união dos pedaços da célula. Só os pixels do próprio glifo vão para o atlas.
5. **Métricas**: a linha de base de cada linha da sheet é medida pela parte de baixo das letras retas (A, B, D, E, H, I…), com ajuste linear entre as linhas. `lineHeight` e `base` vêm daí, incluindo os acentos mais altos.
6. **Atlas**: os glifos são empacotados numa textura de lados potência de 2, com a margem escolhida.
7. **Kerning**: para cada par, compara a borda direita do primeiro com a borda esquerda do segundo em cada linha de pixels em comum, e grava a diferença.

---

## Formato exportado

O `.fnt` é texto no formato BMFont, que o Godot 3.x carrega direto:

```
info face="MinhaFonte" size=199 ...
common lineHeight=199 base=168 scaleW=2048 scaleH=1024 pages=1 packed=0
page id=0 file="MinhaFonte.png"
chars count=157
char id=65 x=... y=... width=... height=... xoffset=0 yoffset=... xadvance=... page=0 chnl=15
...
kernings count=...
kerning first=84 second=65 amount=-40
```

- Cada caractere vira uma linha `char`; minúsculas e aliases repetem o retângulo do glifo original.
- O espaço (32) e o espaço sem quebra (160) são gravados sem imagem, só com o avanço.
- O `page file=` aponta para o PNG pelo nome, então os dois arquivos precisam ficar na **mesma pasta**.

---

## Limitações

- Não reconhece qual letra é qual a partir do desenho; a ordem vem do modelo/campo de texto e é conferida por você.
- O alinhamento vertical parte do princípio de que as letras de uma mesma linha da sheet compartilham a linha de base. Sheets com glifos soltos em posições aleatórias não funcionam bem.
- Com kerning ligado o `.fnt` fica grande (cerca de 780 KB numa sheet com 96 glifos, com mais de 20 mil pares). Aumente "Ignorar kerning menor que" ou desligue o kerning para reduzir.
- Um glifo mais largo que cerca de 1,1× o passo da grade, com uma divisa de célula no meio, pode ser cortado indevidamente como se fossem dois glifos colados.
