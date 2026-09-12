# Suporte à Localização em Português do Brasil (pt-BR) — VoiceInk

Este documento registra a implementação completa da tradução e localização da interface gráfica (GUI) do VoiceInk para **Português do Brasil (pt-BR)**.

---

## 1. Escopo da Tradução

* **Total de Chaves Traduzidas:** 1.167 strings de interface.
* **Strings Regulares:** 1.147 itens (menus, botões, painéis, alertas, dicas e formulários).
* **Regras de Plurais (`stringsdict`):** 20 estruturas de contagem dinâmica (arquivos, modelos, transcrições, dias úteis, etc.).
* **Strings do Sistema (`InfoPlist`):** 7 descrições de permissões do macOS (Microfone, Gravação de Tela, AppleEvents, etc.).

---

## 2. Arquivos Modificados no Repositório

1. **`VoiceInk/App/Configuration/AppLanguagePreference.swift`:**
   * Adicionado o identificador `"pt-BR"` na lista de idiomas nativos (`bundledLanguageIdentifiers`).
   * A interface agora exibe automaticamente a opção **"português (Brasil)"** no seletor de idiomas.

2. **`VoiceInk/Localizable.xcstrings`:**
   * Atualizado com o nó `"pt-BR"` para todas as 1.167 chaves do catálogo moderno de strings da Apple.

3. **`VoiceInk/InfoPlist.xcstrings`:**
   * Adicionadas as traduções para os textos de consentimento de privacidade e permissões do sistema.

---

## 3. Estrutura do Pacote Instalado no macOS

No aplicativo em `/Applications/VoiceInk.app`, foram gerados e compilados os arquivos nativos em:
* `/Applications/VoiceInk.app/Contents/Resources/pt-BR.lproj/Localizable.strings`
* `/Applications/VoiceInk.app/Contents/Resources/pt-BR.lproj/Localizable.stringsdict`
* `/Applications/VoiceInk.app/Contents/Resources/pt-BR.lproj/InfoPlist.strings`
* `/Applications/VoiceInk.app/Contents/Resources/pt.lproj/` (compatibilidade com sistemas configurados para `pt` genérico).

---

## 4. Como Alternar o Idioma

1. Abra o VoiceInk.
2. Acesse o menu **Configurações** (ou pressione `Cmd + ,`).
3. Na seção **Geral**, selecione **Idioma** -> **português (Brasil)** (ou deixe em **Sistema**, que já detectará o português se o macOS estiver no idioma).
4. O app salvará a preferência na chave `AppLanguagePreference` e atualizará o `AppleLanguages`.
