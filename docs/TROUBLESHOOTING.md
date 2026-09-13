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

---

## 6. Armazenamento de Chaves de API de IA e Erro -34018 no Keychain

### Sintoma
Ao inserir uma chave de API de serviços de nuvem ou IA (como Groq, OpenAI, Anthropic, Gemini, Deepgram) na interface gráfica do VoiceInk e clicar em **"Verificar"** / **"Salvar"**, a chave não é persistida (o campo pode voltar a ficar vazio após fechar ou reiniciar o app) e nos logs do macOS / Console aparece o seguinte erro:

```text
Failed to update keychain item for key: groqAPIKey, status: -34018
```
(ou identificadores correlatos como `openAIAPIKey`, `anthropicAPIKey`, etc., acompanhados do código de status `-34018`).

### Causa Técnica
O código original do VoiceInk foi configurado para interagir com a API de Keychain do macOS utilizando parâmetros restritivos:
* `kSecUseDataProtectionKeychain = true`
* `kSecAttrSynchronizable = true`

No ecossistema macOS / iOS, essas duas propriedades ativam o **Data Protection Keychain** com suporte à sincronização de credenciais via iCloud Keychain. No entanto, o subsistema `securityd` do macOS exige obrigatoriamente uma assinatura digital emitida por uma conta corporativa da Apple (*Apple Developer Program*) com um Team ID provisionado e os respectivos *entitlements* (`keychain-access-groups` / `com.apple.developer.keychain-sync`).

Quando o aplicativo é compilado localmente ou re-assinado de forma *ad-hoc* (`codesign -f -s - ...`), o macOS detecta a ausência desses entitlements autorizados e rejeita sumariamente qualquer tentativa de escrita no Keychain, retornando o código de erro:
```text
errSecMissingEntitlement = -34018
```

### Resolução Aplicada
Para garantir total interoperabilidade com assinaturas locais/ad-hoc sem depender de provisionamento Apple Developer pago:

1. **Ajuste na consulta base do Keychain (`KeychainService`):**
   No binário de produção, foi aplicado o patch na rotina `baseQuery` da classe `KeychainService` para construir a query básica de credenciais contendo apenas:
   * `kSecClass` = `kSecClassGenericPassword`
   * `kSecAttrService` = `com.prakashjoshipax.VoiceInk`
   * `kSecAttrAccount` = `keyIdentifier` (ex: `"groqAPIKey"`, `"openAIAPIKey"`)

   Foram suprimidas as chaves restritivas `kSecUseDataProtectionKeychain` e `kSecAttrSynchronizable` (comportamento equivalente ao `#if LOCAL_BUILD` do projeto).

2. **Utilização do Chaveiro de Login Padrão (`login.keychain-db`):**
   Com essa mudança, o VoiceInk passa a gravar as chaves diretamente no **Chaveiro de Login** padrão do usuário (`login.keychain-db`), que:
   * É 100% seguro e criptografado com a chave do usuário logado no macOS;
   * Funciona perfeitamente com binários assinados localmente (*ad-hoc*);
   * Elimina completamente a restrição de entitlements e o erro `-34018`.

> [!NOTE]
> Essa modificação mantém total isolamento por aplicativo no macOS, garantindo que as credenciais fiquem armazenadas de maneira segura e criptografada pelo sistema operacional.

### Como Salvar ou Atualizar Chaves de API

Agora é possível configurar qualquer chave de API diretamente pela interface visual do VoiceInk sem restrições ou necessidade de scripts manuais:

1. Abra o VoiceInk e acesse as **Configurações** (`Cmd + ,`).
2. Vá até a seção **IA / Modelos** (ou **Modelos em Nuvem / Provedores**).
3. Selecione o provedor desejado (**Groq**, **OpenAI**, **Anthropic**, **Google Gemini**, **Deepgram**, etc.).
4. Cole a sua chave de API no campo correspondente.
5. Clique no botão **"Verificar"** (ou **"Salvar"**).
6. O VoiceInk validará o token com a API remota e salvará a chave instantaneamente no Keychain do macOS.

> [!TIP]
> O aplicativo agora persiste as credenciais de qualquer provedor de IA de forma permanente entre reinicializações do sistema operacional.

