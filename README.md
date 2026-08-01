# Codex do Tempo

Um controle de horário para mesas de RPG — dia, hora e minuto em jogo, com avanço rápido, ajuste manual e um registro de eventos carimbado com o timestamp da campanha. Tudo salvo automaticamente, sem precisar de servidor ou banco de dados externo.

## O que é

Ferramenta pensada pra mestres e jogadores que precisam manter o controle da passagem de tempo durante a campanha: quanto tempo dura uma viagem, quando o descanso longo termina, em que dia/hora um evento aconteceu. Um único arquivo HTML, sem dependências de instalação.

## Funcionalidades

- **Dial visual do relógio** — círculo com gradiente de céu que muda de cor conforme a hora do dia (alvorada, manhã, tarde, crepúsculo, noite), com marcador de sol/lua se movendo ao longo das 24h.
- **Avanço rápido de tempo** — botões para +10min, +30min, +1h, +4h, +8h (descanso longo) e "próximo dia".
- **Avanço customizado** — campo para inserir horas e minutos específicos (ex: duração de uma viagem ou ritual).
- **Ajuste manual** — edição direta de dia, hora e minuto, para corrigir ou pular para um ponto da história.
- **Registro de eventos** — anotações de jogo com timestamp automático (dia/hora/minuto), listadas da mais recente para a mais antiga, com opção de remover.
- **Nome da campanha** — campo editável no topo do relógio.
- **Persistência automática e redundante** — todo estado (relógio + eventos + nome da campanha) é salvo a cada alteração em múltiplas camadas locais (`localStorage` + `IndexedDB` + backups rotativos), com salvamento forçado ao trocar de aba ou fechar a página. Funciona offline, em Android, iPhone ou desktop.
- **Backup exportável/importável** — botões para baixar um arquivo `.json` com todo o estado e para importá-lo em outro dispositivo, permitindo levar a campanha de um aparelho para outro.
- **Reinício** — botão para zerar o relógio (volta para Dia 1, 08:00) e apagar todos os eventos, com confirmação antes de executar.

## Como usar

1. Abra o arquivo `codex-do-tempo.html` (ou o artifact, se estiver rodando dentro do Claude).
2. Defina o nome da campanha, se quiser.
3. Use os botões de avanço rápido conforme o tempo passa na mesa, ou o campo de avanço customizado para durações específicas.
4. Anote eventos importantes no campo de registro — cada um guarda o momento exato do jogo em que ocorreu.
5. Feche e reabra quando quiser: o estado é recuperado automaticamente, no mesmo dispositivo e navegador.
6. Para levar os dados para outro celular ou computador, use "exportar backup" e depois "importar backup" no outro aparelho.

## Estrutura técnica

Arquivo único (HTML + CSS + JavaScript embutidos), sem dependências externas de build. A única chamada de rede é a importação da fonte via Google Fonts (`Cormorant Garamond`, `Spectral`, `Space Mono`); sem essa fonte, o navegador cai automaticamente nas fontes de sistema (serif/monospace).

### Persistência

O app funciona 100% no navegador, sem servidor, e salva os dados **localmente no dispositivo** em múltiplas camadas redundantes, sob a chave `codex-do-tempo:state`:

1. **`localStorage`** — leitura/escrita síncrona, suportado por qualquer navegador (Chrome, Safari, Firefox) em Android, iPhone ou desktop.
2. **`IndexedDB`** — camada de backup com mais capacidade e mais resistente a limpezas parciais de dados do navegador.
3. **Rotação de backups** — a cada salvamento, uma cópia carimbada com data/hora é mantida em `localStorage` (as 5 mais recentes), permitindo recuperar o estado caso o registro principal seja corrompido.
4. **Compatibilidade com `window.storage`** — se o app estiver rodando dentro do ambiente de artifacts do Claude, essa API também é usada automaticamente, sem prejuízo às camadas acima.

Ao carregar a página, o app lê todas as fontes disponíveis e usa a mais recente (por timestamp), com fallback automático para as demais em caso de falha ou dado corrompido.

O objeto salvo tem este formato:

```json
{
  "campaignName": "string",
  "day": 1,
  "hour": 8,
  "minute": 0,
  "events": [
    { "id": "ev_...", "day": 1, "hour": 8, "minute": 0, "text": "string", "order": 0 }
  ],
  "_savedAt": 1690000000000,
  "_v": 1
}
```

Como salvamento acontece a cada ação (avanço de tempo, novo evento, edição manual etc.) e também há uma rede de segurança que força o salvamento ao trocar de aba, minimizar o app ou fechar a página (`visibilitychange`, `pagehide`, `beforeunload`), o estado sobrevive a fechar a aba, o navegador, ou reiniciar o aparelho — **contanto que seja reaberto no mesmo navegador, no mesmo dispositivo**, sem limpar dados de navegação.

> **Importante:** por ser uma persistência 100% local (sem backend), os dados **não sincronizam sozinhos entre dispositivos diferentes** — o Codex salvo no celular não aparece automaticamente no notebook, por exemplo.

### Levando os dados para outro dispositivo

Use os botões **"exportar backup (.json)"** e **"importar backup"** no rodapé do app:

- **Exportar** baixa um arquivo `.json` com todo o estado atual (relógio + eventos + nome da campanha).
- **Importar** lê um arquivo `.json` exportado anteriormente e substitui os dados atuais (com confirmação antes de aplicar).

Fluxo típico: exporte no celular → envie o arquivo para si mesmo (e-mail, WhatsApp, Drive, AirDrop) → abra o Codex do Tempo no notebook → importe o arquivo lá.

## Possíveis expansões futuras

- Calendário com meses/estações personalizados (nomes próprios de mundo).
- Fases da lua.
- Múltiplos "relógios" para campanhas paralelas.
- Sincronização automática entre dispositivos via backend próprio (ex: pequeno servidor ou serviço como Firebase/Supabase).

## Licença

Uso livre para fins pessoais e de mesa de jogo.
