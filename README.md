<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FlertAI — Análise de Conversas</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
        :root {
            --roxo: #8B5CF6;
            --verde: #10B981;
            --amarelo: #F59E0B;
            --vermelho: #EF4444;
            --fundo: #0F0F1A;
            --card: #1A1A2E;
            --texto: #E2E8F0;
            --texto-claro: #94A3B8;
        }
        body { background: var(--fundo); min-height: 100vh; padding: 16px; color: var(--texto); }
        .container { max-width: 480px; margin: 0 auto; }
        .cabecalho { text-align: center; margin-bottom: 24px; }
        .cabecalho h1 { font-size: 1.6rem; background: linear-gradient(90deg, #8B5CF6, #EC4899); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .versao { font-size: 0.75rem; color: var(--texto-claro); margin-top: 4px; }

        /* ESFERA INTELIGENTE */
        .esfera-container { display: flex; flex-direction: column; align-items: center; margin: 24px 0; }
        .esfera {
            width: 120px; height: 120px; border-radius: 50%;
            background: radial-gradient(circle at 30% 30%, #A78BFA, #7C3AED, #4C1D95);
            box-shadow: 0 0 40px rgba(139, 92, 246, 0.5);
            transition: all 0.8s ease;
            animation: flutuar 3s ease-in-out infinite;
            position: relative; overflow: hidden;
        }
        .esfera::before {
            content: ''; position: absolute; inset: 0; border-radius: 50%;
            background: radial-gradient(circle at 60% 60%, transparent, rgba(255,255,255,0.15));
        }
        @keyframes flutuar {
            0%,100% { transform: translateY(0); }
            50% { transform: translateY(-8px); }
        }
        .esfera.verde {
            background: radial-gradient(circle at 30% 30%, #6EE7B7, #10B981, #047857);
            box-shadow: 0 0 40px rgba(16, 185, 129, 0.5);
        }
        .esfera.amarelo {
            background: radial-gradient(circle at 30% 30%, #FCD34D, #F59E0B, #D97706);
            box-shadow: 0 0 40px rgba(245, 158, 11, 0.5);
        }
        .esfera.vermelho {
            background: radial-gradient(circle at 30% 30%, #FCA5A5, #EF4444, #DC2626);
            box-shadow: 0 0 40px rgba(239, 68, 68, 0.5);
        }
        .esfera-legenda { text-align: center; margin-top: 12px; font-size: 0.9rem; color: var(--texto-claro); }
        .esfera-status { font-weight: 600; margin-top: 4px; font-size: 1rem; }

        .card { background: var(--card); border-radius: 16px; padding: 20px; margin-bottom: 16px; border: 1px solid rgba(139, 92, 246, 0.15); }
        .card h2 { font-size: 1rem; margin-bottom: 12px; color: var(--roxo); display: flex; align-items: center; gap: 8px; }

        textarea { width: 100%; min-height: 90px; border: 1px solid #2D2D3F; border-radius: 10px; padding: 12px; background: #0F0F1A; color: var(--texto); font-size: 0.95rem; resize: vertical; margin-bottom: 12px; }
        textarea:focus { outline: none; border-color: var(--roxo); }
        
        .upload-area { border: 2px dashed #2D2D3F; border-radius: 12px; padding: 24px; text-align: center; cursor: pointer; margin-bottom: 12px; transition: all 0.3s; }
        .upload-area:hover { border-color: var(--roxo); background: rgba(139, 92, 246, 0.05); }
        #preview-img { max-width: 100%; border-radius: 8px; margin-top: 10px; display: none; }

        button { border: none; border-radius: 10px; font-size: 0.95rem; font-weight: 600; cursor: pointer; transition: all 0.2s; }
        button:active { transform: scale(0.97); }
        .btn-principal { width: 100%; padding: 14px; background: linear-gradient(135deg, var(--roxo), #EC4899); color: white; }
        .btn-principal:hover { opacity: 0.95; }
        .btn-voz { padding: 8px 14px; background: #2D2D3F; color: var(--texto); display: inline-flex; align-items: center; gap: 6px; font-size: 0.85rem; margin-left: 8px; }
        .btn-voz.ativo { background: var(--roxo); }

        .carregando { text-align: center; padding: 30px; display: none; }
        .carregando.visivel { display: block; }
        .spinner { width: 40px; height: 40px; border: 3px solid #2D2D3F; border-top-color: var(--roxo); border-radius: 50%; animation: girar 1s linear infinite; margin: 0 auto 12px; }
        @keyframes girar { to { transform: rotate(360deg); } }

        .resultado { display: none; animation: aparecer 0.5s ease; }
        .resultado.visivel { display: block; }
        @keyframes aparecer { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .bloco { margin-bottom: 18px; }
        .bloco h3 { font-size: 0.9rem; color: var(--texto-claro); margin-bottom: 8px; display: flex; align-items: center; justify-content: space-between; }
        
        .barra-fundo { height: 8px; background: #2D2D3F; border-radius: 4px; margin: 6px 0; }
        .barra-preenchimento { height: 100%; border-radius: 4px; transition: width 1s ease; }
        
        .justificativa { background: rgba(139, 92, 246, 0.05); border-left: 3px solid var(--roxo); padding: 10px; border-radius: 0 8px 8px 0; font-size: 0.9rem; line-height: 1.5; margin-top: 8px; }
        
        .aviso-box { background: rgba(245, 158, 11, 0.08); border-left: 3px solid var(--amarelo); padding: 12px; border-radius: 0 8px 8px 0; font-size: 0.88rem; line-height: 1.5; margin-top: 8px; }
        .aviso-box h4 { color: var(--amarelo); margin-bottom: 6px; font-size: 0.9rem; }

        .resposta-card { padding: 12px; background: #252538; border-radius: 8px; margin-bottom: 8px; cursor: pointer; transition: background 0.2s; border-left: 3px solid var(--roxo); }
        .resposta-card:hover { background: #2D2D4C; }
        .resposta-card strong { display: block; margin-bottom: 4px; font-size: 0.9rem; }
        .resposta-card span { font-size: 0.88rem; color: var(--texto-claro); }

        .copiado { position: fixed; top: 20px; right: 20px; background: var(--verde); color: white; padding: 10px 16px; border-radius: 8px; display: none; z-index: 100; font-size: 0.9rem; }
    </style>
</head>
<body>
    <div class="copiado" id="copiado">✓ Copiado!</div>

    <div class="container">
        <div class="cabecalho">
            <h1>✨ FlertAI</h1>
            <p class="versao">Versão 2026 — Análise com voz e interpretação</p>
        </div>

        <!-- ESFERA INTELIGENTE -->
        <div class="esfera-container">
            <div class="esfera" id="esfera"></div>
            <div class="esfera-legenda">
                <div class="esfera-status" id="esfera-status">Aguardando você...</div>
                <div style="font-size: 0.75rem; margin-top: 4px;">
                    🟢 Interesse • 🟡 Atenção • 🔴 Pouco interesse
                </div>
            </div>
        </div>

        <!-- ENTRADA DE DADOS -->
        <div class="card">
            <h2>📤 Envie a conversa</h2>
            
            <div class="upload-area" id="upload-area">
                <input type="file" id="foto-input" accept="image/*" style="display: none;">
                <div>📷</div>
                <p>Clique para enviar uma foto/screenshot</p>
                <img id="preview-img" alt="Pré-visualização">
            </div>
            
            <textarea id="texto-conversa" placeholder="Ou cole o texto da conversa aqui..."></textarea>
            
            <button class="btn-principal" id="btn-analisar">🔍 Iniciar Análise</button>
        </div>

        <!-- CARREGANDO -->
        <div class="card carregando" id="carregando">
            <div class="spinner"></div>
            <p>Analisando a conversa...</p>
        </div>

        <!-- RESULTADOS -->
        <div class="card resultado" id="resultado">
            <!-- RESUMO -->
            <div class="bloco">
                <h3>📝 Resumo da conversa
                    <button class="btn-voz" id="btn-voz-resumo" onclick="lerTexto('resumo-texto')">🔊 Ouvir</button>
                </h3>
                <p id="resumo-texto" style="line-height: 1.6;"></p>
            </div>

            <!-- ANÁLISE DE INTERESSE -->
            <div class="bloco">
                <h3>📈 Análise de interesse
                    <button class="btn-voz" id="btn-voz-interesse" onclick="lerTexto('texto-interesse')">🔊 Ouvir</button>
                </h3>
                <p style="font-size: 0.9rem; margin-bottom: 6px;">Nível de interesse: <strong id="valor-interesse">--</strong></p>
                <div class="barra-fundo"><div class="barra-preenchimento" id="barra-interesse" style="width: 0%;"></div></div>
                <p id="texto-interesse" class="justificativa"></p>
            </div>

            <!-- O QUE PODE ESTAR ERRADO -->
            <div class="bloco">
                <h3>⚠️ O que pode estar errado na interpretação
                    <button class="btn-voz" id="btn-voz-avisos" onclick="lerTexto('texto-avisos')">🔊 Ouvir</button>
                </h3>
                <div class="aviso-box" id="texto-avisos"></div>
            </div>

            <!-- SUGESTÕES DE RESPOSTA -->
            <div class="bloco">
                <h3>💡 Sugestões de resposta
                    <button class="btn-voz" id="btn-voz-respostas" onclick="lerRespostas()">🔊 Ouvir</button>
                </h3>
                <div id="lista-respostas"></div>
            </div>
        </div>
    </div>

    <script>
        // Elementos
        const esfera = document.getElementById('esfera');
        const esferaStatus = document.getElementById('esfera-status');
        const uploadArea = document.getElementById('upload-area');
        const fotoInput = document.getElementById('foto-input');
        const previewImg = document.getElementById('preview-img');
        const textoConversa = document.getElementById('texto-conversa');
        const btnAnalisar = document.getElementById('btn-analisar');
        const carregando = document.getElementById('carregando');
        const resultado = document.getElementById('resultado');
        const copiado = document.getElementById('copiado');

        let imagemBase64 = null;
        let respostasAtuais = [];
        let sinteseVoz = window.speechSynthesis;

        // Upload de imagem
        uploadArea.addEventListener('click', () => fotoInput.click());
        fotoInput.addEventListener('change', (e) => {
            if (e.target.files[0]) {
                const leitor = new FileReader();
                leitor.onload = (ev) => {
                    previewImg.src = ev.target.result;
                    previewImg.style.display = 'block';
                    imagemBase64 = ev.target.result;
                };
                leitor.readAsDataURL(e.target.files[0]);
            }
        });

        // Mudar cor da esfera
        function mudarEsfera(cor, status) {
            esfera.classList.remove('verde', 'amarelo', 'vermelho');
            if (cor) esfera.classList.add(cor);
            esferaStatus.textContent = status;
        }

        // Leitura em voz
        function lerTexto(idElemento) {
            sinteseVoz.cancel();
            const texto = document.getElementById(idElemento).textContent;
            if (!texto) return;
            const fala = new SpeechSynthesisUtterance(texto);
            fala.lang = 'pt-BR';
            fala.rate = 1.05;
            sinteseVoz.speak(fala);
        }

        function lerRespostas() {
            sinteseVoz.cancel();
            if (respostasAtuais.length === 0) return;
            const textoCompleto = respostasAtuais.map((r, i) => `${i+1}. ${r.nome}: ${r.texto}`).join(' ');
            const fala = new SpeechSynthesisUtterance(textoCompleto);
            fala.lang = 'pt-BR';
            fala.rate = 1.05;
            sinteseVoz.speak(fala);
        }

        // Copiar texto
        function copiarTexto(texto) {
            navigator.clipboard.writeText(texto).then(() => {
                copiado.style.display = 'block';
                setTimeout(() => copiado.style.display = 'none', 1500);
            });
        }

        // Analisar
        btnAnalisar.addEventListener('click', () => {
            const texto = textoConversa.value.trim();
            if (!texto && !imagemBase64) {
                alert('Cole um texto ou envie uma foto da conversa!');
                return;
            }

            // Resetar voz
            sinteseVoz.cancel();
            
            carregando.classList.add('visivel');
            resultado.classList.remove('visivel');
            mudarEsfera('', 'Analisando...');

            setTimeout(() => {
                carregando.classList.remove('visivel');
                const analise = processarAnalise(texto);
                apresentarResultado(analise);
                resultado.classList.add('visivel');
                window.scrollTo({ top: 0, behavior: 'smooth' });
            }, 2000);
        });

        function processarAnalise(texto) {
            const frases = texto.split('\n').filter(f => f.trim());
            let perguntas = 0, emojis = 0, respostasCurtas = 0, entusiasmo = 0;
            let totalMensagens = frases.length;

            frases.forEach(f => {
                if (f.includes('?')) perguntas++;
                const em = f.match(/[😊😍🤭🥰😁❤️😘]/g); if (em) emojis += em.length;
                if (/^(ok|sim|não|talvez|claro|kkk|rs)$/i.test(f.trim())) respostasCurtas++;
                if (/rs|kkk|haha|😆|muito|demais|adorei|legal|bom/i.test(f)) entusiasmo++;
            });

            // Calcular interesse
            let interesse = 50;
            if (perguntas >= 2) interesse += 18;
            if (emojis >= 2) interesse += 12;
            if (entusiasmo >= 1) interesse += 15;
            if (respostasCurtas >= 2) interesse -= 20;
            if (totalMensagens < 3) interesse -= 10;
            interesse = Math.max(15, Math.min(95, interesse));

            // Definir cor e status
            let corEsfera, statusEsfera;
            if (interesse >= 70) {
                corEsfera = 'verde';
                statusEsfera = '✅ Interesse detectado';
            } else if (interesse >= 45) {
                corEsfera = 'amarelo';
                statusEsfera = '⚠️ Atenção: sinal misto';
            } else {
                corEsfera = 'vermelho';
                statusEsfera = '🔴 Sinais de desinteresse';
            }

            // Resumo
            let resumo;
            if (interesse >= 70) resumo = `A conversa está fluindo bem. A pessoa faz perguntas, responde com detalhes e demonstra entusiasmo. Há sinais claros de interesse em manter a conversa.`;
            else if (interesse >= 45) resumo = `O interesse é moderado. A pessoa responde mas sem forte engajamento. Pode estar ocupada ou apenas sendo educada. Continue observando.`;
            else resumo = `A conversa mostra poucos sinais de engajamento. Respostas curtas, poucas perguntas ou demora em responder podem indicar desinteresse ou distração.`;

            // Justificativa
            let justificativa = '';
            if (perguntas >= 2) justificativa += `✅ Faz perguntas — busca manter a conversa\n`;
            if (emojis >= 2) justificativa += `✅ Usa emojis — tom amigável\n`;
            if (entusiasmo >= 1) justificativa += `✅ Demonstra animação\n`;
            if (respostasCurtas >= 2) justificativa += `⚠️ Respostas muito breves\n`;
            if (totalMensagens < 3) justificativa += `⚠️ Poucas mensagens para análise completa\n`;

            // Avisos de interpretação
            let avisos = `<strong>Pode confundir:</strong><br>`;
            avisos += `• Educação com interesse: ser educado não significa necessariamente interesse romântico<br>`;
            avisos += `• Demora com rejeição: pode estar ocupada, não é necessariamente uma recusa<br>`;
            avisos += `• Respostas curtas com desinteresse: pode ser o estilo da pessoa ou estar distraída<br>`;
            avisos += `• Não insistir após recusa: respeite limites explícitos`;

            // Respostas
            let respostas;
            if (interesse >= 70) {
                respostas = [
                    { nome: '💖 Mais carinhoso', texto: 'Adoro conversar com você! Sempre me sinto bem quando a gente fala 😊 Que tal continuarmos mais tarde?' },
                    { nome: '✨ Descontraído', texto: 'Você sempre consegue me fazer sorrir 😄 Tá difícil não querer falar com você!' },
                    { nome: '💬 Direto', texto: 'Gosto muito da nossa conversa. Quer continuar ou prefere descansar agora?' }
                ];
            } else if (interesse >= 45) {
                respostas = [
                    { nome: '💖 Gentil', texto: 'Obrigado por me responder! 😊 Espero que seu dia esteja sendo bom.' },
                    { nome: '✨ Leve', texto: 'Beleza! Então ficamos por aqui por enquanto. Até mais! 😄' },
                    { nome: '🌿 Respeitoso', texto: 'Sem pressa, tá? Fica bem e a gente se fala quando rolar 😊' }
                ];
            } else {
                respostas = [
                    { nome: '💖 Gentil', texto: 'Sem problema! Que você tenha um dia excelente. Fica bem 😊' },
                    { nome: '✨ Leve', texto: 'Beleza! Cuide-se bem e a gente se vê por aí 👋' },
                    { nome: '🌿 Com respeito', texto: 'Entendi. Respeito seu ritmo, sem pressa nenhuma. Até mais!' }
                ];
            }

            respostasAtuais = respostas;

            return {
                corEsfera, statusEsfera, interesse, resumo, justificativa, avisos, respostas
            };
        }

        function apresentarResultado(dados) {
            // Esfera
            mudarEsfera(dados.corEsfera, dados.statusEsfera);

            // Resumo
            document.getElementById('resumo-texto').textContent = dados.resumo;

            // Interesse
            document.getElementById('valor-interesse').textContent = dados.interesse + '%';
            const barra = document.getElementById('barra-interesse');
            barra.style.width = dados.interesse + '%';
            barra.style.background = dados.corEsfera === 'verde' ? '#10B981' : 
                                    dados.corEsfera === 'amarelo' ? '#F59E0B' : '#EF4444';
            document.getElementById('texto-interesse').innerHTML = dados.justificativa.replace(/\n/g, '<br>');

            // Avisos
            document.getElementById('texto-avisos').innerHTML = dados.avisos;

            // Respostas
            const lista = document.getElementById('lista-respostas');
            lista.innerHTML = '';
            dados.respostas.forEach(r => {
                lista.innerHTML += `
                    <div class="resposta-card" onclick="copiarTexto('${r.texto.replace(/'/g, "\\'")}')">
                        <strong>${r.nome}</strong>
                        <span>${r.texto}</span>
                    </div>
                `;
            });
        }
    </script>
</body>
</html>
