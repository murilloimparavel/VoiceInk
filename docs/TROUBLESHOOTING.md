# Guia de Resolução de Problemas (Troubleshooting) — VoiceInk

Este guia reúne diagnósticos práticos, soluções para problemas frequentes de atalhos globais, permissões do macOS, configuração de dispositivos de áudio e escolha de modelos de transcrição local em português.

---

## 1. Resolução do Problema de Atalho Global (Global Shortcut)

### Sintoma
Ao pressionar a tecla de atalho configurada (por exemplo, `Fn`/`🌐` ou atalhos customizados), o VoiceInk não inicia a gravação, não exibe o indicador flutuante na tela ou parece ignorar o teclado.

### Causa Técnica
O VoiceInk utiliza interceptadores globais de eventos de entrada do macOS (`CGEventTap` e monitores globais da API Cocoa) para capturar o pressionamento de teclas mesmo quando outro aplicativo estiver em primeiro plano.

Essas chamadas exigem permissão explícita no subsistema **TCC (Transparency, Consent, and Control)** do macOS, sob a categoria de **Acessibilidade**. Quando o aplicativo passa por modificações locais, atualizações manuais de binário ou re-assinatura ad-hoc (`codesign -f -s - ...`), a assinatura criptográfica do executável é alterada. O macOS detecta a divergência do identificador de código (*designated requirement*) e revoga silenciosamente o privilégio de Acessibilidade — frequentemente mantendo a chave aparentemente "ativada" nos Ajustes do Sistema, mas bloqueando a entrega de eventos reais ao app.

### Procedimento de Correção Passo a Passo

1. **Encerrar o VoiceInk completamente:**
   ```bash
   killall VoiceInk 2>/dev/null || true
   ```

2. **Redefinir a permissão de Acessibilidade do aplicativo no TCC:**
   Abra o Terminal e execute o comando abaixo para limpar os privilégios cacheados do VoiceInk:
   ```bash
   tccutil reset Accessibility com.prakashjoshipax.VoiceInk
   ```

3. **Remover resquícios nos Ajustes do Sistema:**
   * Abra **Ajustes do Sistema > Privacidade e Segurança > Acessibilidade**.
   * Se o item **VoiceInk** ainda estiver listado, selecione-o e clique no botão **remover (`-`)** na base da lista.

4. **Reabrir o VoiceInk e restabelecer a autorização:**
   ```bash
   open -a /Applications/VoiceInk.app
   ```
   * O macOS exibirá uma notificação solicitando permissão de Acessibilidade para o VoiceInk.
   * Clique em **Abrir Ajustes do Sistema** e ative a chave de alternância ao lado do VoiceInk.
   * Se solicitado, confirme a senha de administrador ou autentique com Touch ID.

---

## 2. Modos de Atalho de Gravação

O VoiceInk suporta três modos operacionais de gravação (`RecordingShortcutMode`), adaptando-se a diferentes estilos de trabalho:

| Modo | Identificador | Comportamento | Uso Recomendado |
| :--- | :--- | :--- | :--- |
| **Alternar (Toggle)** | `toggle` | Um toque no atalho inicia a gravação contínua. Outro toque encerra a captura e dispara a transcrição. | Ditados longos, transcrição de reuniões, pensamentos estruturados. |
| **Segurar para Falar (Push-to-Talk)** | `pushToTalk` | A gravação só ocorre enquanto a tecla for mantida pressionada. Ao soltar a tecla, a gravação é interrompida e o texto transcrito é colado. | Frases curtas, mensagens rápidas, respostas em chats. |
| **Híbrido (Hybrid)** | `hybrid` | Um toque rápido (*tap* menor que 0.5s) atua como **Toggle** (grava continuamente até um próximo toque). Manter pressionado por mais de 0.5s atua como **Push-to-Talk** (walkie-talkie). | Máxima versatilidade sem precisar trocar de modo nas configurações. |

### Configuração via Interface Gráfica
1. Abra o VoiceInk e acesse as **Configurações** (`Cmd + ,`).
2. Vá até a seção de **Atalhos**.
3. No campo **Modo do Atalho**, selecione: **Alternar**, **Segurar para Falar** ou **Híbrido**.

### Configuração Direta via Terminal
Você pode alternar o modo padrão diretamente alterando a preferência `primaryRecordingShortcutMode`:

* **Definir como Alternar (Toggle):**
  ```bash
  defaults write com.prakashjoshipax.VoiceInk primaryRecordingShortcutMode -string "toggle"
  ```

* **Definir como Segurar para Falar (Push-to-Talk):**
  ```bash
  defaults write com.prakashjoshipax.VoiceInk primaryRecordingShortcutMode -string "pushToTalk"
  ```

* **Definir como Híbrido (Hybrid):**
  ```bash
  defaults write com.prakashjoshipax.VoiceInk primaryRecordingShortcutMode -string "hybrid"
  ```

* **Consultar o valor atualmente configurado:**
  ```bash
  defaults read com.prakashjoshipax.VoiceInk primaryRecordingShortcutMode
  ```

> [!TIP]
> Caso tenha feito a alteração via terminal com o aplicativo aberto, reinicie o VoiceInk para recarregar os observadores em memória.

---

## 3. Tecla Fn/Globo (`🌐`) vs Atalhos Customizados

### Interceptação Nativa do macOS
Por padrão, nos teclados Apple modernos (MacBook, Magic Keyboard), a tecla `Fn` / Globo (`🌐`) vem associada a comportamentos globais do próprio sistema operacional:
* Abrir o seletor de "Emojis e Símbolos";
* Iniciar o "Ditado do macOS";
* Alternar fontes de entrada de idioma.

Quando o macOS intercepta esse evento em nível de kernel/WindowServer, a tecla pode nunca chegar ao `CGEventTap` do VoiceInk ou causar concorrência direta com o motor de ditado da Apple.

### Como Liberar a Tecla Fn/Globo via Interface Gráfica
1. Abra os **Ajustes do Sistema**.
2. Navegue até **Teclado**.
3. Localize o menu suspenso **"Pressionar tecla 🌐 para"** (ou *"Press 🌐 key to"*).
4. Altere a seleção para **"Não Fazer Nada"** (*Do Nothing*).

Após esse ajuste, a tecla `Fn`/`🌐` poderá ser capturada exclusivamente pelo VoiceInk sem interferências.

### Liberação Programática via Terminal

Caso prefira automatizar a configuração via scripts de setup ou dotfiles, é possível desativar os comportamentos conflitantes do sistema diretamente via linha de comando:

1. **Definir a tecla Fn/Globo para "Não Fazer Nada" (*Do Nothing*):**
   ```bash
   defaults write com.apple.HIToolbox AppleFnUsageType -int 0
   ```
   * O valor `-int 0` configura o subsistema de entrada (`com.apple.HIToolbox`) para ignorar o toque isolado na tecla `Fn`/Globo (`🌐`), evitando a abertura de seletores de emojis ou funções globais do macOS.

2. **Desativar o atalho interno de Ditado do macOS (Hotkey 164):**
   ```bash
   /usr/libexec/PlistBuddy -c "Set :AppleSymbolicHotKeys:164:enabled false" ~/Library/Preferences/com.apple.symbolichotkeys.plist 2>/dev/null || \
   /usr/libexec/PlistBuddy -c "Add :AppleSymbolicHotKeys:164:enabled bool false" ~/Library/Preferences/com.apple.symbolichotkeys.plist
   ```
   * **Por que o Hotkey 164?** O **Hotkey 164** no arquivo `~/Library/Preferences/com.apple.symbolichotkeys.plist` é o atalho simbólico interno do macOS responsável por disparar o Ditado nativo (*Start Dictation*) usando o modificador `Fn` (código de modificador `8388608`).
   * Desativá-lo (`enabled = false`) impede que o sistema operacional abra ou reserve o Ditado da Apple ao pressionar `Fn`, liberando a tecla inteiramente para ser interceptada pelo `CGEventTap` do VoiceInk.

3. **Aplicar as alterações imediatamente sem reiniciar:**
   ```bash
   /System/Library/PrivateFrameworks/SystemAdministration.framework/Resources/activateSettings -u
   ```

* **Consultar o status atual das preferências:**
  ```bash
  # Consultar o comportamento da tecla Fn/Globo (0 = Não Fazer Nada)
  defaults read com.apple.HIToolbox AppleFnUsageType

  # Consultar se o Hotkey 164 (Ditado nativo) está habilitado
  /usr/libexec/PlistBuddy -c "Print :AppleSymbolicHotKeys:164:enabled" ~/Library/Preferences/com.apple.symbolichotkeys.plist 2>/dev/null
  ```

### Recomendações de Atalhos Alternativos Livres de Conflito
Se você utiliza a tecla `Fn`/`🌐` para emojis ou prefere não alterar o comportamento nativo do teclado do macOS, configure um atalho customizado no VoiceInk. As combinações a seguir oferecem ergonomia excelente e baixíssimo índice de conflito:

* **`⌥ + Espaço` (Option + Espaço):** Acesso rápido com o polegar, sem colidir com o Spotlight (`⌘ + Espaço`).
* **`⌃ + Espaço` (Control + Espaço):** Ideal para quem já utiliza `⌥ + Espaço` para launchers como Raycast ou Alfred.
* **Tecla `Option Direita` (`Right Option`):** Funciona como uma tecla única dedicada exclusivamente ao ditado por voz.
* **`⌘ + ⇧ + Espaço` (Command + Shift + Espaço):** Combinação clássica de três teclas de alta precisão.

Para registrar o atalho customizado:
1. Abra **Configurações (`Cmd + ,`) > Atalhos**.
2. Clique no campo **Atalho Principal de Gravação** e pressione a combinação desejada.

---

## 4. Entrada de Áudio e Microfones

### Diagnóstico de Falhas de Áudio
Se a gravação iniciar mas o texto não for gerado (ou retornar em branco), a causa mais frequente é a seleção incorreta do dispositivo de entrada (por exemplo, apontando para uma interface desconectada, placa virtual de streaming ou microfone silenciado).

### Forçar Dispositivo Padrão do Sistema via Terminal
O VoiceInk oferece o modo `System Default`, que acompanha automaticamente a entrada de áudio ativa definida nos Ajustes de Som do macOS (microfone integrado, fones Bluetooth, AirPods, microfone USB externo, etc.).

Para fixar a entrada no padrão do sistema:
```bash
defaults write com.prakashjoshipax.VoiceInk audioInputMode -string "System Default"
```

Para verificar o valor configurado:
```bash
defaults read com.prakashjoshipax.VoiceInk audioInputMode
```

### Testar Manualmente pela Barra de Menus (Menu Bar)
Para isolar se o problema é o atalho de teclado ou o sistema de captura de áudio:

1. Localize o ícone do VoiceInk na **Barra de Menus** superior do macOS (canto superior direito).
2. Clique no ícone para abrir o menu do aplicativo.
3. Clique na opção **"Alternar Gravador"** (*Toggle Recorder*).
4. O mini gravador flutuante será aberto imediatamente:
   * **Se a onda sonora (waveform) oscilar ao falar:** O microfone e o motor de áudio estão funcionando corretamente. Se o atalho físico não funcionava, a falha está nas permissões de Acessibilidade (Seção 1) ou conflito de tecla (Seção 3).
   * **Se a onda sonora permanecer estática (linha reta):** Verifique em **Ajustes do Sistema > Privacidade e Segurança > Microfone** se o VoiceInk possui autorização de captura concedida.

---

## 5. Modelos Locais de Transcrição para Português (pt-BR)

O VoiceInk opera de forma 100% privada e local no macOS, utilizando a aceleração de hardware do Apple Silicon (Neural Engine e Metal) sem enviar dados para a nuvem.

Para transcrições em língua portuguesa, destacam-se duas opções principais:

### 1. Parakeet V3 (`parakeet-tdt-0.6b-v3`) — Máxima Velocidade
* **Arquitetura:** NVIDIA FastConformer-TDT via *FluidAudio* e CoreML.
* **Tamanho do Modelo:** ~494 MB.
* **Desempenho:** Velocidade altíssima (pontuação de velocidade 0.99), latência quase imperceptível.
* **Suporte de Idioma:** Suporte nativo ao português (incluído no catálogo de 25 línguas europeias).
* **Indicação:** Escolha ideal para o dia a dia em computadores Apple Silicon (M1/M2/M3/M4), proporcionando resposta em tempo real quase instantânea após finalizar a fala.

### 2. Whisper Local (via whisper.cpp) — Máxima Acurácia Fonética
O motor Whisper executado nativamente em C++ oferece alta fidelidade na pontuação e vocabulário técnico complexo:

* **Large v3 Turbo (`ggml-large-v3-turbo`):**
  * **Tamanho:** ~1.5 GB.
  * **Consumo de RAM:** ~1.8 GB.
  * **Características:** Acurácia de ponta (0.94) com tempo de processamento muito inferior ao Large v3 tradicional. Excelente interpretação de pontuação e contexto em português.
* **Large v3 Turbo Quantizado (`ggml-large-v3-turbo-q5_0`):**
  * **Tamanho:** ~547 MB.
  * **Consumo de RAM:** ~1.0 GB.
  * **Características:** Versão otimizada com quantização de 5 bits. Mantém praticamente a mesma fidelidade do Turbo completo ocupando menos espaço e consumindo menos memória.
* **Base Multilingual (`ggml-base`):**
  * **Tamanho:** ~142 MB.
  * **Consumo de RAM:** ~0.5 GB.
  * **Características:** Modelo compacto para transcrições rápidas com baixo uso de memória.

### Como Selecionar e Gerenciar Modelos
1. No VoiceInk, abra as **Configurações** (`Cmd + ,`) e clique na aba **Modelos** (ou **Biblioteca de Modelos**).
2. Localize **Parakeet V3** ou o modelo **Whisper** de sua preferência.
3. Clique em **Baixar** para efetuar o download dos pesos do modelo localmente.
4. Após a conclusão do download, defina-o como o modelo ativo de transcrição.
