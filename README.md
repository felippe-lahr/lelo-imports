# Lelo Imports — Landing Page

Landing page de conversão da **Lelo Imports** — peças para carros importados antigos (BMW, Mercedes, Porsche).

## Sobre

Site estático de página única (`index.html`), 100% self-contained: todos os assets
(React, fontes Inter/Sora, imagem hero) já vêm embutidos no próprio HTML. Não há
dependências de rede em tempo de execução — a página é renderizada no navegador a
partir dos dados embutidos.

Chamadas de conversão (CTAs) da página:

- **WhatsApp:** https://wa.me/5511996422226
- **Instagram:** perfil da loja

## Estrutura

```
.
├── index.html      # Landing page completa (single-file, self-contained)
├── .htaccess       # Configuração Apache (Hostinger) — cache e index padrão
└── README.md
```

## Deploy na Hostinger

O site é estático — basta servir o `index.html` a partir do diretório público.

**Via Git (hPanel → Avançado → Git):**

1. Conecte este repositório no hPanel da Hostinger.
2. Defina o diretório de deploy como a pasta pública do domínio (ex.: `public_html`).
3. A cada push na branch de produção, a Hostinger publica os arquivos automaticamente.

**Via upload manual:**

Envie `index.html` e `.htaccess` para `public_html/` pelo Gerenciador de Arquivos
ou FTP.

Ao acessar o domínio, o `index.html` é servido automaticamente como página inicial.
