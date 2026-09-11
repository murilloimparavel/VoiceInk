# Guia de Instalação e Ambiente — VoiceInk

Este documento registra os procedimentos realizados para download do código-fonte, configuração do ambiente e instalação do aplicativo VoiceInk no macOS.

---

## 1. Repositório e Código Fonte

O projeto foi espelhado de forma independente (sem vínculo com o repositório upstream original) e clonado no diretório local de projetos:

* **Repositório GitHub:** `murilloimparavel/VoiceInk`
* **Caminho Local do Código:** `/Users/murilloalves/Projects/VoiceInk`
* **Branch Padrão:** `main`

---

## 2. Instalação do Aplicativo no macOS

O aplicativo executável foi instalado no sistema em:

* **Destino:** `/Applications/VoiceInk.app`
* **Versão Instalada:** `2.13`
* **Bundle Identifier:** `com.prakashjoshipax.VoiceInk`
* **Assinatura e Notarização:** Apple Developer ID (`Prakash Joshi - V6J6A3VWY2`), com ticket de notarização da Apple grampeado (*stapled*).
* **Quarentena do Gatekeeper:** Atributos de quarentena removidos via `xattr -cr`.

---

## 3. Guia para Compilação Local (Opcional)

### Diagnóstico Atual do Ambiente
* **Command Line Tools:** Instalado em `/Library/Developer/CommandLineTools` (`git`, `swift`, `clang` disponíveis).
* **Xcode.app:** Não instalado no momento. A ferramenta `xcodebuild` (necessária para compilar o projeto `.xcodeproj` via terminal) exige a instalação do Xcode completo.

### Passos para Compilar Localmente (se instalar o Xcode no futuro):
1. Instale o **Xcode** através da Mac App Store ou do portal developer.apple.com.
2. Configure o diretório de desenvolvedor ativo com privilégios de administrador:
   `xcode-select -s /Applications/Xcode.app/Contents/Developer`
3. No diretório do projeto (`/Users/murilloalves/Projects/VoiceInk`), execute:
   `make local`
   * O comando compilará a dependência `whisper.xcframework` em `~/VoiceInk-Dependencies`.
   * Gerará o `VoiceInk.app` assinado em modo ad-hoc ou com sua identidade de desenvolvimento Apple em `.local-build/`.

---

## 4. Permissões Necessárias no macOS

Ao abrir o VoiceInk pela primeira vez, o macOS solicitará as seguintes autorizações:
1. **Microfone (`Microphone`):** Obrigatória para captura de voz e ditado.
2. **Acessibilidade (`Accessibility`):** Necessária para o app colar o texto transcrito diretamente no cursor ativo (`Cmd+V`).
3. **Gravação de Tela (`Screen Recording` - Opcional):** Apenas se você desejar ativar o recurso de contexto de tela via OCR local.
