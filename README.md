# Mapa de Abastecimento

Aplicativo de página única (`index.html`, sem build) para programar a navegação fluvial:
combustível dos empurradores e giro das barcaças entre as cidades.

## Módulos

| Módulo | O que faz |
|---|---|
| **Dashboard** | KPIs de consumo, estoque e eficiência. |
| **Premissas** | Empurradores, rotas, **cidades**, **terminais** e **calado médio**. |
| **Médias de Consumo** | Tempo e consumo (L/h) por trecho · empurrador · condição. |
| **Mapa de Abastecimentos** | Grade diária por empurrador: viagem, consumo, abastecimento e estoque de cada combustível. |
| **Mapa de Giros** | Grade diária por **cidade**: viagem, barcaças, volume, navios, terminais e estoque de barcaças. |
| **Abastecimentos / Viagens / Apontamentos / Preços / Importações** | Acompanhamento, finalização e cargas de dados reais. |

Os dois mapas trabalham sobre **a mesma lista de viagens**: mover, criar, editar ou excluir
uma viagem em um deles muda o outro imediatamente.

## Mapa de Giros

Cada cidade vira um bloco de colunas. Ao lado da data fica o **Calado** previsto do rio.

- **Viagem · Barcaças · Volume** — a viagem é plotada na cidade de **destino**, no dia previsto
  de **chegada**, e o chip mostra o **nome do barco** (empurrador); o tooltip traz o trecho,
  a condição, saída, chegada, barcaças, calado e volume. Ao lado vêm a quebra do comboio
  (`20C`, `20V`, `15F` ou, no comboio misto, `10V + 15F`) e o volume que chega.
- **Navio · Barcaças** (cidades que operam navio) — o navio e as barcaças que ele acumula por dia.
  Navio de grão consome barcaça carregada e **devolve vazia**; navio de fertilizante consome
  vazia e **devolve carregada com Fertz**.
- **Estoque** — `0V-4C-15F` com o volume (`12,5k(G)+30k(F)`). Valor negativo = falta de barcaça no dia.
- **Estoque 3º** — barcaças e carga de outros players: contam para o navio, não entram no nosso giro.
- **Terminal: Estoque · Status** (cidade de origem) — cada terminal carrega grão nas vazias e
  descarrega as de Fertz (que viram vazias e carregam ali mesmo). Status: Disponível / Operando / Indisponível.

Interações: clique numa célula vazia de **Viagem** ou **Navio** para lançar (na célula de Viagem
o trecho sugerido é um que **chega** naquela cidade); clique em **Estoque** para informar barcaças
e volume; clique no **Status** para alternar Indisponível (Ctrl+clique abre o período). Arraste os
chips para mudar a data — no Mapa de Giros a viagem é deslocada para **chegar** no dia em que for
solta, mantendo o tempo de trânsito; com **Ctrl** ao soltar, copia.

### Calado (Premissas → Calado Médio)

Um único arquivo traz as duas tabelas:

- `CONF_CALADO_VOLUME` — colunas **Calado (m), Box, Rake, Média**: volume médio de uma barcaça
  por calado (valores intermediários são interpolados).
- `CALC_SIMULACAO` — colunas **Data, Calado Pessimista, Calado Médio, Calado Otimista**: alimenta
  a coluna Calado do mapa. O nome da coluna tem de vir **completo**: uma coluna chamada só
  `Médio`, `Pessimista` ou `Otimista` é ignorada, para a importação não puxar por engano outra
  coluna da planilha que use a mesma palavra.

O calado previsto em **D+1** sugere o calado das barcaças: em Miritituba, pela saída do próximo
comboio; no navio de fertilizante, pelo término previsto da operação.

### Cidades e rotas

As rotas costumam usar siglas (`MIR x STM`). O campo **Apelidos** da cidade liga a sigla ao bloco
— informe `MIR` em Miritituba, `STM` em Santarém, e assim por diante. Quando um ponto de rota não está ligado a
nenhuma cidade, aparece um **⚠ no canto da barra** do Mapa de Giros: o tooltip lista os pontos
que ficam fora do giro e o clique explica como resolver.

### Cidades pulmão (transbordo)

Uma cidade pode servir de **pulmão**: comboios menores levam carga até ela e, de lá, um comboio
maior segue com a carga completa até o destino final. Basta cadastrar a cidade (com o apelido
usado nas rotas) e criar as duas rotas — `MIR x PUL` e `PUL x VDC`. O estoque do pulmão acumula
as barcaças que chegam e as entrega ao comboio que sai.

- A carga **não é recriada no transbordo**: o comboio que sai do pulmão leva o que de fato está
  nas barcaças de lá. Se a viagem for lançada com um calado maior que o da carga que chegou,
  o volume que chega ao destino é o real e o chip da viagem ganha um **⚠** com o valor.
- Chegada e saída **no mesmo dia** funcionam; no dia, as saídas são processadas na ordem da hora.
- Se o comboio maior sair **antes** de a carga chegar, o estoque do pulmão fica **negativo** —
  é o alerta de furo.
- O pulmão **não** deve ser marcado como *Origem*: esse perfil liga os terminais, que carregam
  grão nas barcaças vazias (a cidade passaria a criar carga em vez de só repassá-la).
- Cadastre as **médias de consumo** dos dois trechos: sem elas a viagem não consome combustível
  e a chegada vira apenas "saída + 1 dia".

## Dados

Tudo fica no `localStorage` e, opcionalmente, espelhado numa pasta local (File System Access API).
Use **Exportar dados** / **Importar dados** para mover a base entre máquinas.
