# RankTarefa — App nativo (Capacitor)

Este projeto empacota o RankTarefa (que já é um app estático/PWA em `www/index.html`)
como um app nativo Android/iOS usando o [Capacitor](https://capacitorjs.com).

## Estrutura

```
ranktarefa-capacitor/
├── www/                  ← o app em si (webDir do Capacitor)
│   ├── index.html         (RankTarefa completo — React + Tailwind embutidos)
│   └── manifest.json
├── resources/             ← imagens-fonte para gerar ícones/splash nativos
│   ├── icon.png           (idealmente 1024x1024, sem cantos arredondados)
│   ├── icon-192.png
│   └── icon-512.png
├── package.json
├── capacitor.config.json
└── .gitignore
```

Depois do primeiro `npx cap add android` / `npx cap add ios`, vão aparecer também
as pastas `android/` e `ios/` — são os projetos nativos completos (Android Studio / Xcode).

## Pré-requisitos

- **Node.js** 18+ instalado
- **Android**: Android Studio instalado (com o SDK)
- **iOS**: um Mac com Xcode instalado (não tem como gerar o app iOS sem Mac)

## Passo a passo

### 1. Instalar as dependências

```bash
npm install
```

### 2. Adicionar as plataformas nativas (só precisa fazer uma vez)

```bash
npm run add:android
npm run add:ios      # só se estiver num Mac
```

Isso cria as pastas `android/` e `ios/` com os projetos nativos completos.

### 3. Gerar os ícones e splash screen nativos (opcional, mas recomendado)

Troque `resources/icon.png` por uma imagem quadrada de alta resolução (1024x1024, sem
transparência/cantos arredondados — o Android/iOS arredondam sozinhos) e rode:

```bash
npm run assets
```

Isso gera automaticamente todos os tamanhos de ícone e splash screen que Android e
iOS precisam, dentro de `android/` e `ios/`.

### 4. Sincronizar o código web com os projetos nativos

Sempre que você editar `www/index.html`, rode:

```bash
npm run sync
```

(isso copia os arquivos de `www/` para dentro de `android/` e `ios/`)

### 5. Abrir no Android Studio / Xcode e rodar/gerar o build para a loja

```bash
npm run open:android   # abre o Android Studio
npm run open:ios       # abre o Xcode (só no Mac)
```

A partir daí, o processo de gerar o `.aab` (Google Play) ou arquivar o app (App Store)
é o padrão do Android Studio / Xcode — assinar o app, gerar o build de release, etc.

## Importante antes de publicar

- **`appId`** em `capacitor.config.json` (`br.com.ranktarefa.app`) é o identificador
  único do app nas lojas — **depois de publicado não dá mais pra trocar**. Se você já
  tem um domínio próprio, o ideal é usar o reverso dele (ex: `app.ranktarefa.com` vira
  `com.ranktarefa.app`). Ajuste antes de gerar os projetos nativos.
- O app já detecta quando está rodando dentro do Capacitor e **desliga automaticamente**
  o registro de Service Worker / prompt de instalação PWA (que só fazem sentido no
  navegador) — não precisa mexer em mais nada no código pra isso.
- Os dados continuam salvos só localmente (no armazenamento do próprio app, equivalente
  ao localStorage) — cada professor que instalar o app terá seus próprios dados,
  isolados. Se no futuro migrar para um banco de dados real (Supabase), o app nativo
  também vai poder sincronizar normalmente, já que o WebView do Capacitor tem acesso à
  internet como qualquer app.

## Build automático na nuvem (GitHub Actions — sem instalar nada)

Este projeto já vem com um workflow pronto em `.github/workflows/build-android.yml`
que gera o `.aab` do Android **assinado**, direto na nuvem do GitHub, sem precisar
de Android Studio nem instalar nada no seu computador.

### Passo 1 — Subir o projeto pro GitHub

1. Crie o repositório no GitHub (pode ser privado).
2. Faça upload de **todos** os arquivos e pastas deste projeto — incluindo a pasta
   `.github/` (ela fica escondida, então no upload do site do GitHub marque para
   mostrar arquivos ocultos, ou arraste a pasta inteira do projeto de uma vez).
   **NÃO suba a pasta `keystore/`** (ela nem deveria estar aqui — veja o passo 2).

### Passo 2 — Cadastrar os segredos de assinatura

Você recebeu (fora deste repositório) um arquivo `ranktarefa-release.keystore` e uma
senha. Isso é a "assinatura digital" do seu app — guarde os dois com muito cuidado,
em um lugar seguro (gerenciador de senhas, por exemplo). **Se perder, não tem como
recuperar, e não vai dar mais pra atualizar o app depois de publicado.**

No GitHub, vá em **Settings > Secrets and variables > Actions > New repository secret**
e crie estes 4 segredos:

| Nome do segredo | Valor |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | conteúdo do arquivo `ranktarefa-release.keystore.base64` (cole o texto inteiro) |
| `ANDROID_KEYSTORE_PASSWORD` | a senha que você recebeu |
| `ANDROID_KEY_PASSWORD` | a mesma senha |
| `ANDROID_KEY_ALIAS` | `ranktarefa` |

### Passo 3 — Rodar o build

Vá na aba **Actions** do repositório → clique no workflow **"Build Android (AAB assinado)"**
→ **Run workflow**. Em uns 3-5 minutos ele termina, e o arquivo `.aab` assinado fica
disponível pra download ali mesmo, na seção **Artifacts** da execução.

Esse `.aab` é o arquivo que você sobe direto no Google Play Console.

Sempre que você editar `www/index.html` e enviar (`commit`) pro GitHub, o build roda
de novo sozinho automaticamente.



| Comando | O que faz |
|---|---|
| `npm run sync` | Copia `www/` pros projetos nativos (Android + iOS) |
| `npm run sync:android` | Só Android |
| `npm run sync:ios` | Só iOS |
| `npm run add:android` | Cria o projeto Android (1ª vez) |
| `npm run add:ios` | Cria o projeto iOS (1ª vez) |
| `npm run open:android` | Abre o Android Studio |
| `npm run open:ios` | Abre o Xcode |
| `npm run assets` | Gera ícones/splash nativos a partir de `resources/` |
| `npm run doctor` | Verifica se o ambiente está configurado corretamente |
