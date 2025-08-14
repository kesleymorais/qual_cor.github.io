<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Identificador de Cores Próximas (HTML único)</title>
  <style>
    :root { --bg:#f7f7fb; --card:#fff; --text:#111; --muted:#6b7280; --brand:#4f46e5; }
    *{box-sizing:border-box}
    body{margin:0;background:var(--bg);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Helvetica,Arial,sans-serif}
    .container{max-width:1100px;margin:0 auto;padding:20px}
    h1{font-size:clamp(20px,3vw,28px);margin:0 0 8px}
    .muted{color:var(--muted);font-size:12px}
    .grid{display:grid;gap:16px}
    @media (min-width:900px){.grid-3{grid-template-columns:2fr 1fr}} 
    .card{background:var(--card);border:1px solid #e5e7eb;border-radius:16px;box-shadow:0 2px 10px rgba(0,0,0,.04)}
    .card h2{margin:0 0 10px;font-size:16px}
    .p-16{padding:16px}
    .row{display:grid;gap:16px}
    @media (min-width:700px){.row{grid-template-columns:1fr 1fr}}
    .media{aspect-ratio:16/9;background:#eef0f4;border-radius:12px;display:flex;align-items:center;justify-content:center;overflow:hidden}
    video, img{max-width:100%;height:100%;object-fit:cover}
    img.preview{object-fit:contain;background:#fafafa}
    .btn{display:inline-flex;align-items:center;gap:8px;border:0;background:#e5e7eb;color:#111;padding:10px 14px;border-radius:12px;cursor:pointer}
    .btn.primary{background:var(--brand);color:#fff}
    .btn.success{background:#16a34a;color:#fff}
    .btn:disabled{opacity:.6;cursor:not-allowed}
    input[type="number"], textarea{width:100%;padding:10px 12px;border:1px solid #e5e7eb;border-radius:10px;font:inherit}
    textarea{font-family:ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,monospace;font-size:12px;line-height:1.45}
    .results{display:grid;gap:14px}
    @media (min-width:700px){.results{grid-template-columns:repeat(2,1fr)}}
    @media (min-width:1100px){.results{grid-template-columns:repeat(4,1fr)}}
    .swatch{border:1px solid #e5e7eb;border-radius:14px;overflow:hidden;background:#fff}
    .swatch .color{height:76px}
    .swatch .meta{padding:10px 12px}
    .meta .top{display:flex;justify-content:space-between;align-items:center;gap:8px}
    .badge{font-size:11px;background:#eef0f4;border-radius:999px;padding:2px 8px}
    .bar{height:8px;background:#eef0f4;border-radius:99px;overflow:hidden;margin-top:6px}
    .bar > div{height:100%}
    footer{color:var(--muted);font-size:12px;text-align:center;padding:24px 6px}
    .help{font-size:12px;color:var(--muted)}
    code{background:#eef0f4;padding:1px 6px;border-radius:6px}
  </style>
</head>
<body>
  <div class="container">
    <header style="display:flex;justify-content:space-between;align-items:end;gap:16px;margin-bottom:12px">
      <div>
        <h1>Identificador de Cores Próximas</h1>
        <div class="muted">ΔE (LAB) • roda 100% no navegador • câmera ou upload</div>
      </div>
    </header>

    <div class="grid grid-3">
      <!-- COLUNA IMAGEM -->
      <section class="card p-16">
        <h2>Imagem</h2>
        <div class="row">
          <div>
            <div class="media"><video id="video" playsinline muted></video></div>
            <div style="display:flex;gap:8px;flex-wrap:wrap;margin-top:10px">
              <button id="btnStart" class="btn primary">Ligar câmera</button>
              <button id="btnStop" class="btn">Desligar</button>
              <button id="btnCapture" class="btn success">Capturar</button>
            </div>
          </div>
          <div>
            <div class="media"><img id="preview" class="preview" alt="Prévia"/></div>
            <div style="display:flex;gap:8px;flex-wrap:wrap;margin-top:10px;align-items:center">
              <label class="btn" style="margin:0">
                <input id="fileInput" type="file" accept="image/*" hidden>
                Upload de imagem
              </label>
              <button id="btnAnalyze" class="btn primary">Analisar cores</button>
              <span id="status" class="muted"></span>
            </div>
          </div>
        </div>
        <canvas id="work" hidden></canvas>
      </section>

      <!-- COLUNA PARÂMETROS -->
      <aside class="card p-16">
        <h2>Parâmetros</h2>
        <div style="display:grid;gap:10px">
          <label>
            <div class="muted">Top N cores</div>
            <input id="topN" type="number" min="1" max="32" value="8">
          </label>
          <label>
            <div class="muted">Passo de amostragem (1 = mais preciso)</div>
            <input id="sampleStep" type="number" min="1" max="16" value="4">
          </label>
        </div>
        <div style="margin-top:14px">
          <h2 style="margin:0 0 6px">Paleta (cole aqui sua lista)</h2>
          <textarea id="palette" rows="14"></textarea>
          <div class="help" style="margin-top:8px">
            Formatos aceitos:
            <ul>
              <li>CSV: <code>nome,r,g,b</code> ou <code>nome,#RRGGBB</code></li>
              <li>JSON: <code>[{"name":"Vermelho","r":255,"g":0,"b":0}, ...]</code></li>
            </ul>
          </div>
        </div>
      </aside>
    </div>

    <!-- RESULTADOS -->
    <section class="card p-16" style="margin-top:16px">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
        <h2 style="margin:0">Resultados</h2>
        <div class="muted" id="resCount">—</div>
      </div>
      <div id="results" class="results"></div>
    </section>

    <footer>
      Feito com ❤️ — proximidade calculada em CIE L*a*b* (ΔE76). Ilumine bem ao fotografar.
    </footer>
  </div>

  <script>
    // ------------------------ Estado / elementos ------------------------
    const video = document.getElementById('video');
    const preview = document.getElementById('preview');
    const fileInput = document.getElementById('fileInput');
    const btnStart = document.getElementById('btnStart');
    const btnStop = document.getElementById('btnStop');
    const btnCapture = document.getElementById('btnCapture');
    const btnAnalyze = document.getElementById('btnAnalyze');
    const statusEl = document.getElementById('status');
    const topNEl = document.getElementById('topN');
    const stepEl = document.getElementById('sampleStep');
    const paletteEl = document.getElementById('palette');
    const resultsEl = document.getElementById('results');
    const resCountEl = document.getElementById('resCount');
    const work = document.getElementById('work');

    let stream = null;

    // Paleta padrão
    const DEFAULT_PALETTE = `# Exemplo CSV (name,r,g,b)
Color Name,RGB
25Despertar,248, 235, 187
Tesouro Perdido,248, 229, 179
Gengibre,242, 231, 181
Espuma de Baunilha,243, 241, 229
Lua de Cristal,251, 250, 239
Praia Branca,246, 246, 242
Papel de Seda,237, 236, 227
Papel de Presente,234, 233, 226
Suspiro Caseiro,244, 242, 221
Polpa de Mangostim,252, 247, 225
Som das Ondas,229, 226, 218
Macramê,230, 229, 219
Brilho do Luar,240, 237, 216
Travesseiro de Pena,234, 231, 213
Manteiga de Carité,228, 226, 216
Chapada Diamantina,235, 230, 212
Bem-me-quer,247, 242, 208
Calopsita,219, 218, 215
Teto do Mundo,224, 223, 213
Pena de Ganso,224, 220, 210
Palavras Cruzadas,228, 222, 204
Espuma de Leite,241, 235, 207
Luz de Inverno,238, 233, 201
Espuma Gelada,217, 216, 207
Luzes da Cidade,244, 234, 197
Cordão de Ouro,248, 233, 191
Biscoito de Mantecal,217, 211, 194
Vinho Frisante,234, 227, 192
Sabugo de Milho,219, 217, 200
Pinus,220, 217, 198
Pinoli,245, 228, 187
Pintura Antiga,219, 215, 184
Luz do Sol,221, 215, 183
Calça de Linho,218, 208, 182
Sacola de Lona,207, 200, 183
Mármore,200, 197, 186
Pérola Cintilante,250, 221, 191
Croissant,239, 212, 175
Naturale,249, 208, 181
Pêssego,237, 196, 167
Aquarela Pêssego,251, 205, 173
Nata Fresca,241, 233, 225
Açúcar Cristal,234, 232, 227
Brilho de Estrela,230, 225, 219
Quase Rosa,247, 231, 217
Berço,229, 226, 219
Algodão Egípcio,234, 227, 213
Luar,226, 220, 214
Gelo,230, 234, 225
Palha,223, 212, 188
Ouro Branco,218, 216, 208
Humanidade,244, 228, 202
Esponja do Mar,229, 220, 204
Areia do Deserto,227, 218, 205
Papel Picado,216, 211, 202
Flan de Baunilha,248, 231, 196
Trigo,236, 220, 196
Papel Sedoso,249, 226, 196
Gergelim,242, 217, 199
Duna Maranhense,232, 220, 198
Mormaço,248, 218, 196
Pasta de Amendoim,225, 210, 190
Areia,238, 224, 199
Canjica,233, 209, 184
Marfim Nobre,232, 213, 177
Crômio,209, 208, 202
Tapete de Juta,213, 206, 194
Puma,211, 201, 187
Broto de Feijão,208, 202, 193
Tempestade no Mar,199, 199, 196
Ráfia,218, 207, 183
Inverno Gelado,209, 201, 185
Lamparina,215, 205, 184
Doce de Leite,212, 200, 182
Cocada,215, 200, 177
Andiroba,202, 193, 184
Ovelha,197, 191, 179
Camelo,209, 189, 171
Sorvete de Mamão,254, 190, 154
Cuíca,240, 197, 131
Tropicália,255, 177, 130
Maracá,252, 179, 132
Baião,247, 168, 121
Biscoito-champagne,214, 161, 119
Bronze,208, 147, 117
Damasco Suave,246, 200, 160
Recheio de Avelã,213, 188, 159
Oca,210, 182, 162
Base Soft,240, 196, 159
Néctar de Pêssego,253, 197, 159
Pão de Ló,227, 195, 151
Ninho,199, 181, 156
Frescor da Manhã,253, 212, 148
Capoeira,203, 181, 156
Chocolate,191, 170, 157
Choconhaque,219, 177, 155
Juta,208, 185, 139
Noz-moscada,195, 169, 147
Mel,231, 189, 135
Rosa-queimado,208, 169, 147
Bolo de Nozes,201, 173, 140
Barbante,212, 180, 136
Agreste,219, 168, 139
Chapéu de Sol,216, 164, 136
Granizo,182, 178, 173
Paturi,182, 167, 152
Galho Seco,185, 175, 156
Cavalo-marinho,179, 164, 143
Cadeira de Balanço,170, 156, 146
Folha de Tabaco,179, 154, 135
Estrada Velha,191, 160, 129
Berimbau,180, 155, 132
Passas ao Rum,170, 147, 133
Madagascar,171, 151, 130
Flor-de-anis,195, 154, 129
Vitamina de Papaia,239, 138, 78
12Pétala Laranja,241, 173, 68
Gema Caipira,243, 169, 58
Afrobeat,237, 125, 59
Tijolo,193, 108, 69
General,234, 120, 59
Bumba Meu Boi,242, 159, 109
Bola de Basquete,210, 153, 99
Beira do Cais,161, 131, 102
Terra Arada,151, 120, 93
Chocolate Suíço,161, 119, 82
23Calcita Alaranjada,201, 139, 90
Juba de Leão,166, 130, 87
Sertão,142, 123, 104
Funghi,123, 104, 89
Castanha-do-pará,146, 114, 93
Tronco,121, 101, 89
Doce de Caju,136, 105, 79
Café com Nata,107, 91, 81
Cravo-da-índia,121, 95, 76
Castanho,144, 103, 73
Pinhão,95, 79, 69
Conto de Fadas,243, 212, 222
Delícia de Morango,243, 209, 226
Floco de Neve,245, 245, 245
Lenço de Bolso,234, 232, 231
Espuma Cremosa,242, 233, 231
Rosa Pastel,242, 229, 226
Renda,224, 215, 215
Rosa-bebê,241, 220, 219
Castiçal,232, 213, 214
Sapatilha de Ponta,236, 212, 215
Pingo de Leite,231, 207, 190
Duna,225, 199, 186
Rosa-talco,223, 197, 192
Reco-reco,207, 201, 197
Palito de Picolé,214, 205, 199
Allure,213, 193, 181
Sonho com Nata,216, 189, 173
Jujuba Rosa,240, 196, 213
Rosa-neon,255, 182, 192
Bicho-de-pé,246, 176, 188
Frapê de Pêssego,245, 168, 146
Figo,215, 156, 152
Mousse de Iogurte,245, 168, 172
Bala de Iogurte,243, 162, 155
Fruta-da-condessa,236, 191, 174
Amor Platônico,196, 179, 179
Águia,184, 178, 179
Flor de Cerejeira,196, 178, 174
Rosa Delicado,220, 180, 167
Bala Toffee,220, 182, 166
Rosa Laranja,245, 185, 170
Árvore dos Sonhos,207, 173, 164
Primorosa,240, 178, 150
21Meia-luz,197, 167, 172
Açucena,234, 174, 157
Rosa Giz,186, 159, 156
Argila,210, 158, 140
Bombom de Licor,183, 161, 154
Suéter,157, 148, 151
Canoa Quebrada,168, 146, 135
Tapete de Crochê,156, 139, 135
Vela Aromatizada,181, 143, 150
Surpresa de Amora,178, 144, 154
Cinza-espacial,140, 137, 136
Grão de Café,166, 140, 132
Saia Justa,187, 133, 128
Goiaba,222, 118, 124
Melancia,220, 92, 88
Telha Nova,200, 99, 85
Paixão Inspiradora,175, 45, 59
Flamboyant,202, 79, 67
Vermelho-amor,205, 66, 68
Valentino,198, 38, 45
Cereja,168, 41, 57
Rosa-comportado,189, 134, 137
Lenha,153, 131, 119
Fim de tarde,200, 131, 116
Híbisco Selvagem,162, 120, 139
Bossa Nova,204, 130, 127
Tapeçaria,166, 125, 116
17Cortina de Teatro,157, 108, 117
18Terra Roxa,187, 117, 97
Terra batida,137, 103, 88
Anis estrelado,157, 94, 118
Batom Malva,138, 95, 112
Marrom-glacê,159, 93, 89
Hena,145, 88, 82
Mogno Brasileiro,129, 84, 67
Camu-camu,160, 79, 88
Vinho Tinto,132, 73, 85
Batom de Cereja,145, 67, 86
Rubi,145, 68, 76
Telhado,143, 76, 65
Desejo,132, 48, 56
Bolsa de Veludo,125, 39, 50
Banco da Praça,146, 124, 114
Gaita,124, 112, 116
Geleia de Amora,137, 104, 114
Borra de Café,114, 94, 84
Festival de Teatro,106, 96, 99
Cabruca,86, 68, 60
Favo de Baunilha,83, 76, 73
Chocolate Meio Amargo,97, 79, 72
Aceto Balsâmico,82, 73, 76
Carvão,77, 74, 73
Canção de Ninar,242, 213, 228
Chuva de Pétalas,222, 207, 229
Flor-de-pessegueiro,239, 205, 228
Pedra Tapume,244, 243, 248
Chá de Rosas,232, 225, 234
Rosa Suave,237, 228, 235
Linha de Costura,235, 218, 234
Chá de Melissa,219, 217, 235
Espaço Sideral,204, 206, 219
Fio Metalizado,202, 204, 209
Pelo de Coelho,202, 200, 205
Mangaratiba,197, 197, 205
Essência de Lavanda,188, 192, 208
Rosa-biquíni,239, 188, 213
Rosa luminoso,232, 174, 212
Botão de Rosa,193, 151, 176
Flor-de-gerânio,198, 163, 213
Suave Lilás,224, 196, 223
Novelo de Lã,194, 183, 194
Vinícola,185, 181, 194
Ikebana,215, 189, 225
Frapê de Uva,206, 178, 191
Violeta-vapor,196, 177, 194
Templo dos Deuses,188, 185, 213
Bodega,180, 171, 182
Água de Cheiro,188, 190, 223
Cortina Infantil,175, 171, 200
Sofia,167, 174, 199
Sachê de Lavanda,167, 179, 213
Flauta Doce,172, 161, 171
Yoga,162, 147, 166
Fim de Estação,171, 149, 166
Violeta-queimado,163, 150, 167
Entardecer,160, 150, 177
Maravilha,205, 101, 159
Violeta-africana,194, 141, 192
Uva Tempranillo,145, 157, 209
Rosa da vez,181, 125, 160
Supernova,118, 122, 136
Flor-do-maracujá,166, 129, 160
Desfile de Moda,152, 124, 146
Quaresmeira,167, 130, 183
Ipê-rosa,192, 113, 158
Estrela Radiante,145, 131, 197
Lírio-africano,121, 113, 150
Alamanda Roxa,151, 106, 141
Campânula,129, 108, 153
Glamour,139, 83, 143
Geleia de Uva,124, 67, 106
Roxo-rústico,71, 52, 106
Túnel do Tempo,109, 113, 129
Marinho Profundo,71, 76, 107
15Índigo Blue,61, 65, 95
Céu Sereno,199, 226, 237
Banho Gelado,202, 223, 236
Lago de Gelo,205, 231, 246
Pingo de Chuva,187, 216, 221
Hortência Suave,191, 213, 232
Azul-bebê,183, 213, 229
Manhã do Ártico,228, 232, 235
Cafuné,236, 246, 249
Azaleia Branca,226, 230, 236
Creme de Limão,233, 245, 248
Água Fresca,213, 233, 236
Imã de geladeira,219, 232, 235
Chave de Fenda,208, 212, 214
Caravela do mar,211, 235, 241
Arrecifes,203, 214, 224
Tranquilidade,197, 209, 220
Azul-polar,198, 215, 229
Pirineus,189, 209, 214
Respiração Profunda,189, 201, 220
Chuva de Prata,197, 202, 209
Névoa Intensa,189, 195, 199
Vapor de Água,191, 196, 200
Asteroide,186, 192, 196
Luz da Lua,184, 195, 204
Hortência Lilás,183, 191, 205
Céu Azul,159, 207, 233
Vida Azul,145, 187, 223
Cor do Céu,147, 180, 215
Riacho Doce,148, 177, 218
Azul Celestial,130, 208, 240
Ilhas Gregas,117, 188, 225
Azul-guache,126, 175, 201
Geleiras,174, 219, 235
Lago Congelado,175, 199, 210
Céu de Inverno,190, 202, 228
Opalina,170, 208, 232
Azul-etéreo,165, 181, 197
Amanhecer,164, 194, 215
Rio Ebro,146, 182, 193
Céu Nublado,155, 175, 191
Marine,125, 136, 187
Pepita de Bismuto,173, 175, 180
24Conforto,163, 174, 176
Prata Bali,151, 157, 163
Golfinho,144, 158, 168
Lago Grey,133, 164, 173
Pedra,128, 137, 139
Azul-rei,98, 171, 219
Brilho Solitário,50, 159, 206
14Curaçau Blue,32, 165, 214
Borboleta Azul,61, 136, 196
Oceano Pacífico,32, 93, 145
Gelatina de Tutti Frutti,78, 177, 205
Veludo Azul,117, 145, 166
Trovoada,127, 136, 156
Rio Reno,99, 133, 142
Cruzeiro Marítimo,110, 125, 161
Centáurea-azul,101, 144, 202
Planeta Azul,85, 142, 187
Jeans Lavado,96, 125, 153
Lago Sarmiento,49, 148, 174
Azul-precioso,0, 110, 138
Giz de Cera,17, 60, 117
Pó de Grafite,108, 115, 123
Taça Real,110, 118, 120
Baleia-azul,94, 105, 121
Anoitecer,78, 105, 130
Meia-noite,65, 69, 85
Marinheiro,61, 67, 78
Preto Absoluto,40, 41, 43
Azul-marinho,60, 72, 92
Calça Jeans,43, 79, 112
Calmaria,200, 233, 231
Refresco de Menta,205, 238, 236
Orvalho do Campo,229, 234, 236
Luz da Manhã,198, 220, 222
Brilho Vítreo,188, 199, 198
Oceano Índico,144, 207, 205
Mar Mediterrâneo,121, 184, 179
Horizonte,163, 186, 183
Brisa da Montanha,165, 203, 199
Bolinha de Gude,147, 191, 184
20Mantra,150, 168, 167
Vaso de Cerâmica,121, 168, 161
Martim-pescador,0, 150, 163
Topo do Mundo,0, 129, 143
Mar Verde,0, 121, 111
Lagoa Cristalina,117, 177, 176
Verde-nostalgia,72, 192, 178
Ilha Deserta,100, 160, 156
Apanhador de Sonhos,92, 154, 149
Azul-petróleo,10, 116, 124
Suspiro Mentolado,202, 238, 221
Verde-preguiça,202, 222, 196
Sapatilha Verde,189, 234, 224
Fábula,212, 226, 191
Drinque Refrescante,202, 235, 205
Igarapé,193, 233, 215
Verde-cristalino,185, 231, 220
Sereia do Mar,187, 231, 207
Seixo Branco,245, 246, 246
Branco Neve,253, 255, 248
Esqui na Neve,233, 235, 232
Creme Delicado,227, 234, 231
Gelato de hortelã,241, 246, 237
Suspiro,229, 233, 229
Frescor de Menta,223, 230, 225
Verde-água,215, 231, 215
Mousse de Limão,223, 231, 210
Limonada Suíça,212, 228, 218
Frescor da Natureza,216, 225, 214
Verde-lavado,217, 225, 206
Chá de Menta,215, 232, 212
Esperança,209, 225, 212
Verde Sublime,217, 228, 210
Contemplacão,192, 208, 207
Regata,207, 219, 195
Verde-silencioso,184, 209, 195
Chuva de Verão,195, 200, 198
Jardim de Pedras,193, 200, 195
Caipirinha de Limão,185, 199, 187
Lago Frias,181, 225, 179
Verde-piscina,155, 222, 182
Rio Límpido,148, 218, 182
Verde Cartolina,182, 230, 175
Balão de Festa,188, 232, 168
Pistache,160, 200, 154
Suco de Clorofila,175, 223, 154
Alto-mar,128, 202, 182
16Água-marinha,131, 185, 156
Verde-catamarã,178, 207, 197
Passeio Ecológico,184, 205, 181
Fundo do Mar,189, 197, 177
Verde-boêmia,177, 214, 192
Equilíbrio,176, 225, 210
Sereia,176, 229, 195
Rio Bonito,173, 224, 200
Soda Italiana,164, 226, 211
Flor do Antúrio,206, 221, 160
Verde-orgânico,170, 197, 166
Ervas Finas,188, 211, 164
Verde-pastel,159, 183, 156
Ponte estaiada,171, 178, 169
Inox,166, 170, 165
Planta Fantasma,163, 174, 159
Cashmere,159, 175, 150
Babosa,152, 166, 139
Nanquim,137, 139, 136
Eucalipto,133, 164, 133
Verde-chá,189, 208, 87
Verde-trevo,101, 180, 117
Verde-neon,167, 198, 67
Sombra Verde,169, 189, 54
Verde-cítrico,162, 187, 37
Brilho da Esmeralda,93, 136, 60
Folha de Menta,196, 214, 107
Palmeira,137, 156, 120
Erva Melissa,132, 162, 121
Mochila de Aventura,122, 131, 116
Trilha na Mata,109, 124, 95
Espinafre,104, 124, 88
Montanha Encantada,82, 93, 80
Patativa,80, 81, 79
Chenile,223, 231, 193
Banhos Termais,236, 235, 229
Papel-crepom,230, 230, 225
Erva-doce,235, 234, 218
Praia Deserta,225, 224, 205
Doce de Limão,240, 236, 192
Capim-santo,231, 230, 192
Limão com Mel,229, 227, 191
Limonada,212, 215, 189
Folha de Uva,204, 209, 174
Rio Paíne,209, 210, 199
Cheiro-verde,184, 184, 116
Folha Seca,190, 193, 166
Acelga,203, 205, 150
Verdite,177, 177, 125
Torre de Pisa,178, 178, 163
Quero-quero,176, 175, 154
Elefante,162, 162, 157
Acrópole,163, 163, 144
Chicória,169, 170, 128
Umbu do Sertão,153, 154, 131
Pântano,157, 154, 131
Salgueiro,181, 171, 122
Oásis,179, 168, 112
Cítrico,211, 213, 106
Selva,142, 140, 116
Kiwi,164, 164, 91
Tempero Sírio,155, 152, 66
Capim-seco,124, 120, 92
Branco Puro,250, 249, 247
Geada,233, 233, 232
Sambaqui,219, 218, 219
Foto Retrô,205, 204, 202
Banho de Platina,191, 192, 190
Pena de Prata,182, 182, 181
Fantasia Prateada,169, 169, 167
Nuvem de Chuva,156, 156, 156
Cinza Natural,146, 145, 145
Cinza Clássico,135, 135, 136
Cinza Tecnológico,125, 125, 124
Granito Nobre,104, 103, 103
Carvão Mineral,81, 81, 81
Aventurina Preta,48, 48, 48
Canção de Ninar,242, 213, 228
Banho Gelado,202, 223, 236
Verde-preguiça,202, 222, 196
Fábula,212, 226, 191
Drinque Refrescante,202, 235, 205
Chuva de Pétalas,222, 207, 229
Hortência Suave,191, 213, 232
Flor-de-pessegueiro,239, 205, 228
Pedra Tapume,244, 243, 248
Seixo Branco,245, 246, 246
Branco Neve,253, 255, 248
Esqui na Neve,233, 235, 232
Manhã do Ártico,228, 232, 235
Gelato de hortelã,241, 246, 237
Suspiro,229, 233, 229
Azaleia Branca,226, 230, 236
Chá de Rosas,232, 225, 234
Rosa Suave,237, 228, 235
Frescor de Menta,223, 230, 225
Verde-água,215, 231, 215
Mousse de Limão,223, 231, 210
Limonada Suíça,212, 228, 218
Frescor da Natureza,216, 225, 214
Chave de Fenda,208, 212, 214
Verde-lavado,217, 225, 206
Chá de Menta,215, 232, 212
Linha de Costura,235, 218, 234
Esperança,209, 225, 212
Verde Sublime,217, 228, 210
Chá de Melissa,219, 217, 235
Arrecifes,203, 214, 224
Tranquilidade,197, 209, 220
Regata,207, 219, 195
Azul-polar,198, 215, 229
Espaço Sideral,204, 206, 219
Respiração Profunda,189, 201, 220
Fio Metalizado,202, 204, 209
Pelo de Coelho,202, 200, 205
Chuva de Prata,197, 202, 209
Névoa Intensa,189, 195, 199
Mangaratiba,197, 197, 205
Jardim de Pedras,193, 200, 195
Vapor de Água,191, 196, 200
Asteroide,186, 192, 196
Luz da Lua,184, 195, 204
Essência de Lavanda,188, 192, 208
Hortência Lilás,183, 191, 205
Caipirinha de Limão,185, 199, 187
Rosa-biquíni,239, 188, 213
Lago Frias,181, 225, 179
Céu Azul,159, 207, 233
Verde-piscina,155, 222, 182
Rosa luminoso,232, 174, 212
Verde Cartolina,182, 230, 175
Balão de Festa,188, 232, 168
Vida Azul,145, 187, 223
Pistache,160, 200, 154
Cor do Céu,147, 180, 215
Suco de Clorofila,175, 223, 154
Botão de Rosa,193, 151, 176
Riacho Doce,148, 177, 218
Flor-de-gerânio,198, 163, 213
Ilhas Gregas,117, 188, 225
16Água-marinha,131, 185, 156
Passeio Ecológico,184, 205, 181
Fundo do Mar,189, 197, 177
Suave Lilás,224, 196, 223
Novelo de Lã,194, 183, 194
Céu de Inverno,190, 202, 228
Sereia,176, 229, 195
Vinícola,185, 181, 194
Opalina,170, 208, 232
Ikebana,215, 189, 225
Frapê de Uva,206, 178, 191
Violeta-vapor,196, 177, 194
Flor do Antúrio,206, 221, 160
Verde-orgânico,170, 197, 166
Templo dos Deuses,188, 185, 213
Ervas Finas,188, 211, 164
Bodega,180, 171, 182
Água de Cheiro,188, 190, 223
Azul-etéreo,165, 181, 197
Amanhecer,164, 194, 215
Cortina Infantil,175, 171, 200
Verde-pastel,159, 183, 156
Sofia,167, 174, 199
Sachê de Lavanda,167, 179, 213
Céu Nublado,155, 175, 191
Marine,125, 136, 187
Pepita de Bismuto,173, 175, 180
Ponte estaiada,171, 178, 169
Inox,166, 170, 165
Planta Fantasma,163, 174, 159
Prata Bali,151, 157, 163
Flauta Doce,172, 161, 171
Cashmere,159, 175, 150
Golfinho,144, 158, 168
Yoga,162, 147, 166
Fim de Estação,171, 149, 166
Babosa,152, 166, 139
Violeta-queimado,163, 150, 167
Nanquim,137, 139, 136
Entardecer,160, 150, 177
Eucalipto,133, 164, 133
Azul-rei,98, 171, 219
Verde-chá,189, 208, 87
Brilho Solitário,50, 159, 206
Verde-trevo,101, 180, 117
Maravilha,205, 101, 159
Borboleta Azul,61, 136, 196
Verde-neon,167, 198, 67
Sombra Verde,169, 189, 54
Verde-cítrico,162, 187, 37
Brilho da Esmeralda,93, 136, 60
Oceano Pacífico,32, 93, 145
Folha de Menta,196, 214, 107
Violeta-africana,194, 141, 192
Uva Tempranillo,145, 157, 209
Palmeira,137, 156, 120
Rosa da vez,181, 125, 160
Pérola Cintilante,250, 221, 191
Conto de Fadas,243, 212, 222
Croissant,239, 212, 175
Naturale,249, 208, 181
Gengibre,242, 231, 181
Delícia de Morango,243, 209, 226
Pêssego,237, 196, 167
Aquarela Pêssego,251, 205, 173
Espuma de Baunilha,243, 241, 229
Floco de Neve,245, 245, 245
Papel de Seda,237, 236, 227
Papel de Presente,234, 233, 226
Nata Fresca,241, 233, 225
Açúcar Cristal,234, 232, 227
Lenço de Bolso,234, 232, 231
Espuma Cremosa,242, 233, 231
Brilho de Estrela,230, 225, 219
Quase Rosa,247, 231, 217
Berço,229, 226, 219
Algodão Egípcio,234, 227, 213
Travesseiro de Pena,234, 231, 213
Rosa Pastel,242, 229, 226
Luar,226, 220, 214
Gelo,230, 234, 225
Calopsita,219, 218, 215
Pena de Ganso,224, 220, 210
Palha,223, 212, 188
Ouro Branco,218, 216, 208
Humanidade,244, 228, 202
Esponja do Mar,229, 220, 204
Areia do Deserto,227, 218, 205
Renda,224, 215, 215
Luz de Inverno,238, 233, 201
Espuma Gelada,217, 216, 207
Rosa-bebê,241, 220, 219
Papel Picado,216, 211, 202
Flan de Baunilha,248, 231, 196
Trigo,236, 220, 196
Papel Sedoso,249, 226, 196
Gergelim,242, 217, 199
Duna Maranhense,232, 220, 198
Mormaço,248, 218, 196
Castiçal,232, 213, 214
Pinus,220, 217, 198
Sapatilha de Ponta,236, 212, 215
Pinoli,245, 228, 187
Pasta de Amendoim,225, 210, 190
Pingo de Leite,231, 207, 190
Areia,238, 224, 199
Canjica,233, 209, 184
Marfim Nobre,232, 213, 177
Luz do Sol,221, 215, 183
Duna,225, 199, 186
Rosa-talco,223, 197, 192
Crômio,209, 208, 202
Tapete de Juta,213, 206, 194
Reco-reco,207, 201, 197
Palito de Picolé,214, 205, 199
Puma,211, 201, 187
Broto de Feijão,208, 202, 193
Tempestade no Mar,199, 199, 196
Ráfia,218, 207, 183
Inverno Gelado,209, 201, 185
Lamparina,215, 205, 184
Doce de Leite,212, 200, 182
Calça de Linho,218, 208, 182
Sacola de Lona,207, 200, 183
Mármore,200, 197, 186
Cocada,215, 200, 177
Andiroba,202, 193, 184
Prata,192, 192, 186
Ovelha,197, 191, 179
Allure,213, 193, 181
Sonho com Nata,216, 189, 173
Camelo,209, 189, 171
Jujuba Rosa,240, 196, 213
Rosa-neon,255, 182, 192
Sorvete de Mamão,254, 190, 154
Bicho-de-pé,246, 176, 188
Grão-de-bico,250, 214, 140
Cuíca,240, 197, 131
Ambrosia,249, 218, 138
Tropicália,255, 177, 130
Maracá,252, 179, 132
Frapê de Pêssego,245, 168, 146
Figo,215, 156, 152
Mousse de Iogurte,245, 168, 172
Bala de Iogurte,243, 162, 155
Pó de Gengibre,209, 185, 123
Sequilho,255, 217, 115
Baião,247, 168, 121
Seiva de Cajueiro,228, 193, 119
Biscoito-champagne,214, 161, 119
Cacau da Bahia,255, 209, 113
Bronze,208, 147, 117
Fruta-da-condessa,236, 191, 174
Damasco Suave,246, 200, 160
Cristal de Pirita,203, 193, 166
Cinza-urbano,185, 182, 172
Amor Platônico,196, 179, 179
Águia,184, 178, 179
Sisal,201, 193, 165
Recheio de Avelã,213, 188, 159
Oca,210, 182, 162
Flor de Cerejeira,196, 178, 174
Base Soft,240, 196, 159
Rosa Delicado,220, 180, 167
Néctar de Pêssego,253, 197, 159
Bala Toffee,220, 182, 166
Pão de Ló,227, 195, 151
Ninho,199, 181, 156
Frescor da Manhã,253, 212, 148
Capoeira,203, 181, 156
Rosa Laranja,245, 185, 170
Cerrado,203, 188, 156
Chocolate,191, 170, 157
Árvore dos Sonhos,207, 173, 164
Choconhaque,219, 177, 155
Tecelagem,195, 185, 149
Primorosa,240, 178, 150

# Ou JSON:
# [{"name":"Minha Cor","r":123,"g":45,"b":67}, ...]
# Ou CSV com HEX: nome,#RRGGBB`;
    paletteEl.value = DEFAULT_PALETTE;

    // ------------------------ Câmera ------------------------
    btnStart.addEventListener('click', async () => {
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' }, audio: false });
        video.srcObject = stream;
        await video.play();
      } catch (e) {
        alert('Não foi possível acessar a câmera: ' + (e && e.message ? e.message : e));
      }
    });

    btnStop.addEventListener('click', () => {
      if (stream) stream.getTracks().forEach(t => t.stop());
      stream = null;
      video.pause();
      video.srcObject = null;
    });

    btnCapture.addEventListener('click', () => {
      if (!video.videoWidth) return;
      const { width, height } = contain(video.videoWidth, video.videoHeight, 900, 900);
      work.width = width; work.height = height;
      const ctx = work.getContext('2d');
      ctx.drawImage(video, 0, 0, width, height);
      work.toBlob(b => { preview.src = URL.createObjectURL(b); }, 'image/jpeg', 0.9);
    });

    // ------------------------ Upload ------------------------
    fileInput.addEventListener('change', e => {
      const f = e.target.files && e.target.files[0];
      if (!f) return;
      preview.src = URL.createObjectURL(f);
    });

    // ------------------------ Analisar ------------------------
    btnAnalyze.addEventListener('click', async () => {
      if (!preview.src) { alert('Carregue uma imagem ou capture uma foto.'); return; }
      statusEl.textContent = 'Processando...';
      resultsEl.innerHTML = '';
      resCountEl.textContent = '—';
      await analyze();
      statusEl.textContent = '';
    });

    async function analyze(){
      const img = await loadImage(preview.src);
      // Redimensiona para acelerar
      const MAX_DIM = 600;
      const { width, height } = contain(img.width, img.height, MAX_DIM, MAX_DIM);
      work.width = width; work.height = height;
      const ctx = work.getContext('2d');
      ctx.drawImage(img, 0, 0, width, height);
      const { data } = ctx.getImageData(0, 0, width, height);

      const palette = parsePalette(paletteEl.value).map(p => ({...p, lab: rgbToLab(p.r,p.g,p.b)}));
      if (!palette.length){ alert('Sua paleta está vazia ou inválida.'); return; }
      const counts = new Array(palette.length).fill(0);
      const distSums = new Array(palette.length).fill(0);

      const step = Math.max(1, Math.min(16, +stepEl.value||4));
      for (let y=0; y<height; y+=step){
        for (let x=0; x<width; x+=step){
          const i = (y*width + x)*4;
          const a = data[i+3]; if (a < 10) continue;
          const r = data[i], g = data[i+1], b = data[i+2];
          const lab = rgbToLab(r,g,b);
          let bestI = 0, bestD = Infinity;
          for (let k=0;k<palette.length;k++){
            const d = deltaE76(lab, palette[k].lab);
            if (d < bestD){ bestD = d; bestI = k; }
          }
          counts[bestI]++;
          distSums[bestI]+=bestD;
        }
      }

      const total = counts.reduce((a,b)=>a+b,0)||1;
      const res = palette.map((p,idx)=>({
        name:p.name, r:p.r, g:p.g, b:p.b, hex:rgbToHex(p.r,p.g,p.b),
        count:counts[idx], percentage:counts[idx]/total*100,
        meanDeltaE: counts[idx] ? distSums[idx]/counts[idx] : Infinity
      })).filter(r=>r.count>0)
        .sort((a,b)=> b.count - a.count || a.meanDeltaE - b.meanDeltaE)
        .slice(0, Math.max(1, Math.min(32, +topNEl.value||8)));

      renderResults(res);
    }

    function renderResults(list){
      resCountEl.textContent = list.length + ' cores';
      const frag = document.createDocumentFragment();
      list.forEach((r, idx)=>{
        const card = document.createElement('div');
        card.className = 'swatch';
        const color = document.createElement('div');
        color.className = 'color';
        color.style.background = r.hex;
        const meta = document.createElement('div');
        meta.className = 'meta';
        const top = document.createElement('div');
        top.className = 'top';
        const name = document.createElement('div');
        name.textContent = r.name;
        name.title = r.name;
        name.style.fontWeight = '600';
        name.style.overflow = 'hidden';
        name.style.textOverflow = 'ellipsis';
        name.style.whiteSpace = 'nowrap';
        const rank = document.createElement('div');
        rank.className = 'badge';
        rank.textContent = '#' + (idx+1);
        top.appendChild(name);
        top.appendChild(rank);
        const small = document.createElement('div');
        small.className = 'muted';
        small.style.marginTop = '4px';
        small.textContent = r.hex.toUpperCase() + ' • rgb(' + r.r + ',' + r.g + ',' + r.b + ')';
        const share = document.createElement('div');
        share.className = 'muted';
        share.textContent = 'Participação: ' + r.percentage.toFixed(1) + '%  •  ΔE médio: ' + (isFinite(r.meanDeltaE)? r.meanDeltaE.toFixed(1): '—');
        const bar = document.createElement('div');
        bar.className = 'bar';
        const barIn = document.createElement('div');
        barIn.style.width = Math.max(1, r.percentage) + '%';
        barIn.style.background = r.hex;
        bar.appendChild(barIn);
        meta.appendChild(top);
        meta.appendChild(small);
        meta.appendChild(share);
        meta.appendChild(bar);
        card.appendChild(color);
        card.appendChild(meta);
        frag.appendChild(card);
      });
      resultsEl.innerHTML = '';
      resultsEl.appendChild(frag);
    }

    // ------------------------ Utils ------------------------
    function contain(sw, sh, mw, mh){
      const k = Math.min(mw/sw, mh/sh);
      return { width: Math.max(1, Math.round(sw*k)), height: Math.max(1, Math.round(sh*k)) };
    }

    function loadImage(url){
      return new Promise((resolve,reject)=>{
        const img = new Image();
        img.crossOrigin = 'anonymous';
        img.onload = ()=>resolve(img);
        img.onerror = reject;
        img.src = url;
      });
    }

    function clamp255(n){ return Math.max(0, Math.min(255, Math.round(n))); }
    function rgbToHex(r,g,b){
      const to2 = n => n.toString(16).padStart(2,'0');
      return '#' + to2(clamp255(r)) + to2(clamp255(g)) + to2(clamp255(b));
    }
    function hexToRgb(hex){
      const h = (hex||'').replace('#','').trim();
      if(!/^([0-9a-fA-F]{6})$/.test(h)) return {r:NaN,g:NaN,b:NaN};
      return { r:parseInt(h.slice(0,2),16), g:parseInt(h.slice(2,4),16), b:parseInt(h.slice(4,6),16) };
    }

    function parsePalette(text){
      const lines = text.split(/\n+/).map(l=>l.trim()).filter(l=>l && !l.startsWith('#'));
      const joined = lines.join('\n');
      // tenta JSON
      try{
        const arr = JSON.parse(joined);
        if(Array.isArray(arr)){
          return arr.map(normalizePalette).filter(Boolean);
        }
      }catch{}
      // CSV
      const out = [];
      for(const line of lines){
        const parts = line.split(/\s*,\s*/);
        if(parts.length < 2) continue;
        const name = parts[0];
        if(parts[1].startsWith('#')){
          const {r,g,b} = hexToRgb(parts[1]);
          if(Number.isFinite(r)) out.push({name, r, g, b});
          continue;
        }
        if(parts.length >= 4){
          const r = clamp255(+parts[1]);
          const g = clamp255(+parts[2]);
          const b = clamp255(+parts[3]);
          if([r,g,b].every(v=>Number.isFinite(v))) out.push({name,r,g,b});
        }
      }
      return out;
    }

    function normalizePalette(o){
      if(!o || typeof o.name !== 'string') return null;
      if(typeof o.hex === 'string'){
        const {r,g,b} = hexToRgb(o.hex);
        return { name:o.name, r, g, b };
      }
      if(['r','g','b'].every(k=> typeof o[k]==='number')){
        return { name:o.name, r:clamp255(o.r), g:clamp255(o.g), b:clamp255(o.b)};
      }
      return null;
    }

    // sRGB -> XYZ -> LAB (D65)
    function rgbToXyz(r,g,b){
      let R=r/255, G=g/255, B=b/255;
      R = R<=0.04045 ? R/12.92 : Math.pow((R+0.055)/1.055,2.4);
      G = G<=0.04045 ? G/12.92 : Math.pow((G+0.055)/1.055,2.4);
      B = B<=0.04045 ? B/12.92 : Math.pow((B+0.055)/1.055,2.4);
      const X = R*0.4124564 + G*0.3575761 + B*0.1804375;
      const Y = R*0.2126729 + G*0.7151522 + B*0.0721750;
      const Z = R*0.0193339 + G*0.1191920 + B*0.9503041;
      return {X,Y,Z};
    }
    function xyzToLab(X,Y,Z){
      const Xr = 0.95047, Yr = 1.00000, Zr = 1.08883;
      let x = X/Xr, y = Y/Yr, z = Z/Zr;
      const f = t => (t > 0.008856) ? Math.cbrt(t) : (7.787*t + 16/116);
      const fx=f(x), fy=f(y), fz=f(z);
      const L=116*fy-16; const a=500*(fx-fy); const b=200*(fy-fz);
      return {L,a,b};
    }
    function rgbToLab(r,g,b){
      const {X,Y,Z} = rgbToXyz(r,g,b); return xyzToLab(X,Y,Z);
    }
    function deltaE76(l1,l2){
      const dL=l1.L-l2.L, da=l1.a-l2.a, db=l1.b-l2.b;
      return Math.sqrt(dL*dL + da*da + db*db);
    }
  </script>
</body>
</html>
