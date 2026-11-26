# 🌸 Passo a passo para atualizar o site da Fran

São 3 partes, **nesta ordem**:
1. **Google Apps Script**: pasta de fotos nova + senha protegida (feito na conta Google da Fran)
2. **GitHub**: enviar o site novo e as imagens
3. **Testar**

> Por que a ordem importa? O site novo confere a senha no Apps Script. Se o site for publicado
> antes do passo 1, a Fran não consegue entrar na Área dela, mas o catálogo continua funcionando
> normalmente para as clientes.

---

## PARTE 1 — Google Apps Script (conta da Fran)

Entre na **conta Google da Fran**, abra a planilha e vá em **Extensões → Apps Script**.

### 1.1 Trocar a pasta de fotos do Drive
O link da pasta **não fica no site**: quem salva as fotos é o Apps Script. Procure no código
uma linha parecida com esta (use **Ctrl+F** e busque por `getFolderById`):

```javascript
DriveApp.getFolderById("ID-ANTIGO-AQUI")
```

Troque o que está entre aspas pelo ID da pasta nova:

```
1Pl8FeHMMrnAe_-1u49AcHOpBVkzw_4uQ
```

### 1.2 Deixar as fotos visíveis para as clientes
No Google Drive, clique com o botão direito na pasta de fotos → **Compartilhar** →
em "Acesso geral" escolha **Qualquer pessoa com o link** → **Leitor**.
Sem isso, as fotos dos produtos aparecem quebradas no catálogo.

### 1.3 Guardar a senha nova
1. No Apps Script, clique na engrenagem **⚙️ Configurações do projeto** (menu à esquerda).
2. Role até **Propriedades do script** → **Adicionar propriedade do script**.
3. Propriedade: `ADMIN_PASSWORD` · Valor: *(uma senha NOVA)*.
4. **Salvar propriedades do script**.

> ⚠️ Não reutilize a senha antiga (`Fr@n|9489|`). Ela continua visível no histórico do GitHub.

### 1.4 Conferir a senha no `doPost`
Volte ao editor (ícone `< >`), encontre a função `doPost(e)` e cole este bloco
**logo no início dela**:

```javascript
function doPost(e) {
  // ===== PROTEÇÃO POR SENHA (início) =====
  var dados = JSON.parse(e.postData.contents);
  var senhaCorreta = PropertiesService.getScriptProperties().getProperty('ADMIN_PASSWORD');

  if (!senhaCorreta || dados.senha !== senhaCorreta) {
    Utilities.sleep(1500); // deixa tentativas de adivinhar a senha mais lentas
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: 'Senha incorreta.' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  if (dados.acao === 'login') {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  delete dados.senha; // a senha nunca vai para a planilha
  // ===== PROTEÇÃO POR SENHA (fim) =====

  // ... aqui continua o código que já existia ...
}
```

**Atenção:** o código antigo provavelmente já tem uma linha como
`var data = JSON.parse(e.postData.contents);` (o nome pode variar).
- Se o nome for **`dados`**: apague a linha antiga.
- Se for **outro nome** (ex.: `data`): troque a linha antiga por `var data = dados;`.

Clique em **💾 Salvar**.

### 1.5 Publicar o Apps Script pela conta da Fran
Como agora a planilha e a pasta estão na conta dela, o ideal é que o script **rode como ela**:

1. **Implantar → Nova implantação**.
2. Tipo (engrenagem ⚙️): **App da Web**.
3. **Executar como:** *Eu (e-mail da Fran)* · **Quem pode acessar:** *Qualquer pessoa*.
4. **Implantar** → autorize o acesso. Se aparecer "O Google não verificou este app", clique em
   **Avançado → Acessar (não seguro)**. É normal, porque o script é dela mesma.
5. **Copie a URL do App da Web** (termina em `/exec`).

Depois abra o `index.html` e, na **linha 510**, troque a URL antiga pela nova:

```javascript
const API_URL = "https://script.google.com/macros/s/COLE-A-URL-NOVA-AQUI/exec";
```

> Se a implantação atual já for da conta da Fran, dá para manter o mesmo link:
> **Implantar → Gerenciar implantações → ✏️ → Versão: Nova versão → Implantar**.
> Nesse caso não precisa mexer na `API_URL`.

---

## PARTE 2 — GitHub

No repositório do site, deixe os arquivos exatamente assim:

```
index.html              ← substituir pelo novo
img/
   banner.webp          ← banner do topo
   logo.webp            ← logo
   fran.webp            ← foto da Fran (seção "Sobre")
   fran-rosto.webp      ← rostinho da Fran (selo do banner)
```

Pelo site do GitHub:
1. Abra o repositório → **Add file → Upload files**.
2. Arraste o `index.html` **e a pasta `img` inteira**.
3. Embaixo, clique em **Commit changes**.
4. Se tinha enviado as imagens da versão anterior (`img/fran.jpg` e `img/fran-unhas.jpg`),
   pode apagá-las: abra cada uma → ícone 🗑️ → **Commit changes**. Elas não são mais usadas.

Espere **1 a 2 minutos** para o site atualizar e abra com **Ctrl + F5**
(no celular, feche e abra a aba de novo) para não ver a versão antiga guardada.

---

## PARTE 3 — Testar ✅

- [ ] O banner, a logo e a foto da Fran aparecem.
- [ ] Os produtos carregam (se não carregarem, confira a `API_URL` do passo 1.5).
- [ ] O botão do **Instagram** abre o perfil @fransilva_adesivos.
- [ ] O botão do **WhatsApp** abre a conversa com a Fran.
- [ ] Adicionar um produto na sacolinha e enviar o pedido pelo WhatsApp.
- [ ] Rodapé → **🔒 Área da Fran** → uma senha errada é recusada e a senha nova entra.
- [ ] Cadastrar um produto de teste: a foto deve aparecer na **pasta nova** do Drive e no catálogo.
      Depois é só apagar a linha de teste na planilha.

Se algum item falhar, me mande um print do erro (ou o código do Apps Script) que eu ajudo! 💕
