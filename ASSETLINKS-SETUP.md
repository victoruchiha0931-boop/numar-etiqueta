# Como corrigir: App abrindo como site (em vez de app nativo)

O app está abrindo com barra de URL porque o arquivo **assetlinks.json** não está configurado. Sem ele, o Android não confia no app e usa "Custom Tabs" (como site) em vez de TWA (como app nativo).

---

## Passo 1: Obter o SHA-256 do seu keystore

No PowerShell, execute (use a **senha** que você definiu ao criar o keystore):

```powershell
& "C:\Users\Victor\.bubblewrap\jdk\jdk-17.0.11+9\bin\keytool.exe" -list -v -keystore "C:\Users\Victor\Desktop\etiqueta\android.keystore" -alias android
```

Procure na saída algo como:

```
Impressão do certificado SHA256: AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA
```

**Copie esse valor** (o SHA-256, em formato com dois pontos `:`).

---

## Passo 2: Gerar o assetlinks.json

### Opção A – Via Bubblewrap (recomendado)

1. Abra `twa-manifest.json` e adicione o fingerprint no array `fingerprints`:

```json
"fingerprints": [
  {
    "name": "SHA256",
    "value": "AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA"
  }
]
```

2. Execute:

```powershell
cd C:\Users\Victor\Desktop\etiqueta
bubblewrap fingerprint generateAssetLinks
```

O arquivo será gerado em `.well-known/assetlinks.json`.

### Opção B – Manual

1. Copie `.well-known/assetlinks.json.template` para `.well-known/assetlinks.json`
2. Substitua `COLOQUE_SEU_SHA256_AQUI` pelo seu SHA-256
3. **Importante:** remova os dois pontos (`:`) do fingerprint. Exemplo:
   - Original: `AA:BB:CC:DD:...`
   - No JSON: `AABBCCDD...`

---

## Passo 3: Publicar o assetlinks.json no GitHub Pages

O arquivo **precisa estar em** `https://victoruchiha0931-boop.github.io/.well-known/assetlinks.json`  
(ou seja, no **domínio raiz**, não dentro de `/numar-etiqueta/`).

### Como fazer:

1. Crie um repositório chamado **`victoruchiha0931-boop.github.io`** no GitHub.
2. Dentro dele, crie a pasta `.well-known`
3. Coloque o `assetlinks.json` dentro de `.well-known`
4. Faça commit e push
5. No repositório, vá em **Settings → Pages** e configure a branch `main` para GitHub Pages (se ainda não estiver)

### Estrutura esperada:

```
victoruchiha0931-boop.github.io/
└── .well-known/
    └── assetlinks.json
```

### Verificar:

Depois de publicar, confira se acessa sem erro 404:

https://victoruchiha0931-boop.github.io/.well-known/assetlinks.json

---

## Passo 4: Limpar cache no celular e testar

1. **Configurações** → **Apps** → **NUMAR**
2. **Armazenamento** → **Limpar cache** e **Limpar dados**
3. Ou: desinstale o app e instale novamente o APK
4. Abra o app novamente – deve abrir em modo app nativo, sem barra de URL

---

## Resumo

| Problema | Causa |
|----------|--------|
| App abre como site (com barra de URL) | Sem assetlinks.json ou configurado incorretamente |
| Android usa Custom Tabs | Não encontra ou não confia no assetlinks.json |
| Solução | assetlinks.json correto em `/.well-known/` no domínio raiz |
