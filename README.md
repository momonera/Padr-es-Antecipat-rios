# Padr-es-Antecipat-rios

KbName('UnifyKeyNames');
escapeKey = KbName('ESCAPE');
space     = KbName('space');

% Inicialização do Psychtoolbox
PsychDefaultSetup(2);
Screen('Preference', 'SkipSyncTests', 1); % Ignorar testes de sincronização TIRAR QUANDO FOR EXPERIMENTAR
% Captura o nome do voluntário
nome = inputdlg({'Nome do voluntário'}, '', [1, 25]);
nome = char(nome);
centrox=960;
centroy=540;
dist_tela = 57;
% Abre uma janela em modo fullscreen
screenNumber = max(Screen('Screens'));

[window, rect] =  PsychImaging('OpenWindow', screenNumber, [0 0 0]); % Fundo preto

[scr_xsize_mm, scr_ysize_mm] = Screen('DisplaySize', window);

% distancia da tela em cm
scr_xsize_cm = scr_xsize_mm/10;
scr_ysize_cm = scr_ysize_mm/10;

% distancia da tela em pixel
scr_xsize = rect(3);
scr_ysize = rect(4);

tamfix = 5 ; % Tamanho do ponto de fixação
tamestim = 10; % Tamanho do estímulo (círculo)
dist_dva = 8; % Distância entre estímulos/pistas e o centro


dist = round(dva2pix(dist_tela, scr_xsize_cm, scr_xsize, dist_dva));

tame = round(dva2pix(dist_tela, scr_xsize_cm, scr_xsize, tamestim));

tamp = round(dva2pix(dist_tela, scr_xsize_cm, scr_xsize, tamfix));

% Define o tamanho da fonte
fontSize = 50; % Aumente o valor para aumentar o tamanho da letra
Screen('TextSize', window, fontSize);
% Instruções
instructionText = ['Instrução para o experimento:\n' ...
    'Quando um estímulo aparecer, aperte uma tecla o mais rápido possível.\n' ...
    'Tente não apertar a tecla antecipadamente.\n\n' ...
    'Aperte qualquer tecla para iniciar.'];
% Exibe as instruções na tela
DrawFormattedText(window, instructionText, 'center', 'center', [255 255 255]); % Texto branco
Screen('Flip', window); % Atualiza a tela
% Espera até o usuário pressionar uma tecla
KbWait;
%
continuar = 'Aperte qualquer tecla para iniciar';
% Exibe as instruções na tela
DrawFormattedText(window, continuar, 'center', 'center', [255 255 255]); % Texto branco
Screen('Flip', window); % Atualiza a tela
KbWait; % Espera até o usuário pressionar uma tecla
% Contagem regressiva
for i = 5:-1:1
    % Desenha o número na tela
    DrawFormattedText(window, num2str(i), 'center', 'center', [255 255 255]); % Texto branco
    % Atualiza a tela para mostrar o número
    Screen('Flip', window);
    WaitSecs(1);
end

% Coordenadas para os elementos
ponfix=[centrox, centroy]; % Ponto de fixação
estimesq = [centrox - dist, centroy]; % Lado esquerdo
estimdir = [centrox + dist, centroy]; % Lado direito
% Configurações do experimento
blocos = 6;
tentativas = 5;
est=1;
% Cabeçalho do arquivo de dados
fileID = fopen("prototipo_psychtoolbox.txt", "a");
fprintf(fileID, "nome;blocos;tentativa;tpf;pp;pe;soa;validade;rt\n");
fclose(fileID);
% Loop do experimento
for bloco = 1:blocos % Itera pelos blocos (1 a 6)
    for tent = 1:tentativas % Itera pelas tentativas (1 a 5)
        % Blocos 1 a 3: Padrão fixo 300 > 700 > 500
        if bloco <= 3
            if est == 1
                tpf = 300;
            elseif est == 2
                tpf = 700;
            else
                tpf = 500;
            end
            est = est + 1;
            if est > 3
                est = 1; % Reinicia
            end
        else
            % Blocos 4 a 6: Sorteio condicional
            if tpf == 300 || tpf == 500
                tpfposs = [300, 700];
                tpf = tpfposs(randi([1, 2]));
            elseif tpf == 700
                tpf = 500; % Após 700, sempre vem 500
            end
        end

        Screen('BlendFunction', window, GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
        % Exibe ponto de fixação
        Screen('DrawDots', window, ponfix, tamfix, [000 000 000], [], 2);
        Screen('DrawDots', window, ponfix, tamfix, [255 255 255], [], 2); % Ponto de fixação
        Screen('Flip', window);

        WaitSecs(tpf / 1000);

        % Determinação da pista (pp)
        pp = randi(2);
        Screen('DrawDots', window, ponfix, tamfix, [255 255 255], [], 2); % Ponto de fixação
        if pp == 1
            % Pista à esquerda
            Screen('DrawLine', window, [255 255 255], ponfix(1), ponfix(2) - 20, ponfix(1) - 20, ponfix(2), 2);
            Screen('DrawLine', window, [255 255 255], ponfix(1), ponfix(2) + 20, ponfix(1) - 20, ponfix(2), 2);
        else
            % Pista à direita
            Screen('DrawLine', window, [255 255 255], ponfix(1), ponfix(2) - 20, ponfix(1) + 20, ponfix(2), 2);
            Screen('DrawLine', window, [255 255 255], ponfix(1), ponfix(2) + 20, ponfix(1) + 20, ponfix(2), 2);
        end
        Screen('Flip', window);
        WaitSecs(0.2);

        Screen('BlendFunction', window, GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
        % Voltar à fixação
        Screen('DrawDots', window, ponfix, tamfix, [255 255 255], [], 2);
        Screen('Flip', window);

        % SOA aleatório
        soa = (randi(4) * 0.2);
        WaitSecs(soa);

        % Determinação do estímulo (pe)
        if rand() >= 0.1
            pe = pp; % Estímulo válido
        else
            pe = 3 - pp; % Estímulo inválido
        end

        % Exibição do estímulo
        Screen('DrawDots', window, ponfix, tamfix, [000 000 000], [], 2); % Ponto de fixação
        if pe == 1
            Screen('BlendFunction', window, GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
            % Estímulo à esquerda
            Screen('FillOval', window, [255 255 255], CenterRectOnPointd([0 0 tamestim tamestim], estimesq(1), estimesq(2)));
            Screen('DrawDots', window, ponfix, tamfix, [255 255 255], [], 2); % Ponto de fixação
        else
            Screen('BlendFunction', window, GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
            % Estímulo à direita
            Screen('FillOval', window, [255 255 255], CenterRectOnPointd([0 0 tamestim tamestim], estimdir(1), estimdir(2)));
            Screen('DrawDots', window, ponfix, tamfix, [255 255 255], [], 2); % Ponto de fixação
        end
        Screen('Flip', window);
        % Medição do tempo de resposta (rt)
        startTime = GetSecs;
        KbWait; % Aguarda resposta
        rt = GetSecs - startTime;
        % Determinação da validade
        validade_num = (pp == pe);
        % Salvando os resultados no arquivo
        fileID = fopen("prototipo_psychtoolbox.txt", "a");
        fprintf(fileID, "%s;%d;%d;%.3f;%d;%d;%.3f;%d;%.3f\n", nome, bloco, tent, tpf, pp, pe, soa, validade_num, rt);

        
        respToBeMade = true;
            while respToBeMade
                [~,~,keyCode] = KbCheck;
                if keyCode(escapeKey)
                    ShowCursor;
                    sca;
                    return
                elseif keyCode(space)
                    respToBeMade = false;
                end
            end

    end
    WaitSecs(0.5);
    DrawFormattedText(window, 'Fim do bloco. Aperte qualquer tecla para continuar.', 'center', 'center', [255 255 255]); % Texto branco
    Screen('Flip', window); % Atualiza a tela
    % Espera até o usuário pressionar uma tecla
    KbWait;
    for i = 5:-1:1
        % Desenha o número na tela
        DrawFormattedText(window, num2str(i), 'center', 'center', [255 255 255]); % Texto branco
        % Atualiza a tela para mostrar o número
        Screen('Flip', window);
        WaitSecs(1);
    end

end


% Fechar a janela
Screen('CloseAll');
% Análise dos dados após o experimento
fileID = fopen('prototipo_psychtoolbox.txt', 'r');
formatSpec = '%s%d%d%f%d%d%f%d%f';
data = textscan(fileID, formatSpec, 'Delimiter', ';', 'HeaderLines', 1);
fclose(fileID);
