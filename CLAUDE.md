# Lelo Imports — Landing Page

Landing page de conversão da Lelo Imports (peças para importados antigos e novos:
BMW, Mercedes, Porsche, Audi, Alfa Romeo). Site estático, sem build, sem testes,
sem dependências de pacote — o deploy é copiar arquivos para o servidor.

## Estrutura

```
index.html    # a página inteira, gerada por um bundler (ver abaixo)
.htaccess     # Apache/Hostinger: DirectoryIndex, gzip, cache, headers
imagens/      # fotos servidas do disco em runtime — precisam ir para o deploy
README.md
```

## `index.html` é um artefato gerado — leia antes de editar

O arquivo tem 398 linhas e ~2,6 MB porque quase tudo está embutido. A estrutura
real são quatro blocos `<script type="__bundler/*">` no final do arquivo:

| Linha | Bloco            | Conteúdo                                                      |
|-------|------------------|---------------------------------------------------------------|
| 384   | `manifest`       | JSON `uuid → {mime, compressed, data}` — 14 recursos em base64 |
| 388   | `ext_resources`  | mapeia ids lógicos (`heroImg`, URLs do unpkg) → uuid           |
| 392   | `page_order`     | `[]`                                                           |
| 396   | `template`       | **o HTML real da página**, como string JSON-escapada (~80 KB)  |

O `manifest` carrega React 18.3.1 + react-dom (gzip+base64) e 9 fontes woff2
(Inter/Sora). As URLs `unpkg.com` e `fonts.googleapis.com` que aparecem no
arquivo são só ids e `preconnect` residuais — **nada disso é buscado na rede em
runtime**. As imagens, porém, são carregadas do disco (ver "Imagens").

O HTML das primeiras ~380 linhas é o loader do bundler (splash, thumbnail,
`window.__resources`). Não mexa nele.

### Como editar a página

Editar a linha 396 à mão não funciona: é uma string JSON de 80 KB numa única
linha. Decodifique, altere, re-encode. Este roundtrip foi verificado como
**byte-idêntico** — use exatamente esta codificação:

```python
import json
p = 'index.html'
lines = open(p, encoding='utf-8').read().split('\n')

i = next(k for k, x in enumerate(lines)
         if '<script type="__bundler/template">' in x) + 1
tpl = json.loads(lines[i])

tpl = tpl.replace('...antigo...', '...novo...')   # sua alteração aqui

lines[i] = json.dumps(tpl, ensure_ascii=False).replace('</', r'<\/')
open(p, 'w', encoding='utf-8').write('\n'.join(lines))
```

O `.replace('</', '<\\/')` **não é opcional**: sem ele, o primeiro `</script>`
dentro do template fecha o `<script>` que o contém e a página quebra por
inteiro. Para os blocos JSON (manifest, ext_resources, page_order) a regra é a
mesma, mas com `separators=(',', ':')`.

### Verificação obrigatória depois de qualquer edição

Não existe teste automatizado. Este é o check mínimo — rode sempre:

```bash
python3 -c "
import json
l = open('index.html', encoding='utf-8').read().split('\n')
for bloco in ['manifest', 'ext_resources', 'page_order', 'template']:
    i = next(k for k, x in enumerate(l) if f'<script type=\"__bundler/{bloco}\">' in x)
    json.loads(l[i + 1])
print('ok: os 4 blocos do bundler continuam parseáveis')
"
```

Depois abra o `index.html` no navegador e confira o que você mexeu. Um JSON
inválido em qualquer um dos blocos deixa a página em branco, sem erro visível.

## Imagens

O site **não** é self-contained: as fotos são referenciadas como
`imagens/*.webp` e carregadas do disco em runtime. São 15 referências (hero,
logos, e 12 cards de categoria). Se `imagens/` não subir junto no deploy, a
página quebra visualmente.

Convenção da pasta:

- `imagens/originais/` e `imagens/categorias/originais/` — arquivos originais
  (`.png`/`.jpg`), preservados e **não** usados pelo site.
- `imagens/*.webp` e `imagens/categorias/*.webp` — versões otimizadas, ~800 px,
  que o site realmente usa.

Ao adicionar uma foto: guarde a original em `originais/`, gere o `.webp` (~800 px)
e referencie o `.webp` no template. Os cards de categoria são exibidos com
recorte `cover` em 1:1 — prefira imagens quadradas. Categorias sem foto caem num
SVG ilustrativo padrão; `imagens/categorias/README.md` tem a tabela de status.

Não há `cwebp` nem ImageMagick neste ambiente. Para converter, instale Pillow
(`pip install Pillow`) ou peça os `.webp` já prontos.

## Deploy (Hostinger)

Site estático servido do diretório público. Suba **`index.html`, `.htaccess` e a
pasta `imagens/`** para `public_html/` — via Git no hPanel (Avançado → Git) ou
por FTP/Gerenciador de Arquivos.

O `.htaccess` faz `index.html` revalidar sempre (`access plus 0 seconds`), então
atualizações aparecem no ato; imagens e fontes têm cache longo. Se uma imagem
trocar de conteúdo mantendo o nome, o cache de 1 mês segura a versão antiga —
renomeie o arquivo nesse caso.

## Conteúdo que não pode quebrar

Os CTAs são o objetivo da página inteira. Ao editar, confirme que continuam
intactos:

- WhatsApp: `https://wa.me/5511996422226` (botões + bubble flutuante)
- Instagram: `https://instagram.com`
- `<title>` e `<meta name="description">` dentro do `<helmet>` no template

## Dívidas conhecidas

- **Hero morto no manifest (~2,0 MB, 79% do arquivo).** O commit d7c69c2 trocou
  o hero para `imagens/hero-2.webp` (53 KB), mas o PNG antigo continua no
  manifest sob o uuid `3b715297…` — 1,52 MB decodificados que todo visitante
  baixa sem usar. Junto com ele sobrou um script que faz polling de 60 ms por
  10 s procurando `#lelo-hero-car`, um elemento que não existe mais no template.
  Remover ambos derruba o arquivo de 2,6 MB para ~550 KB.
- **README desatualizado.** Diz que o site é "100% self-contained" sem
  dependências de rede e que basta subir `index.html` + `.htaccess`. As imagens
  vêm do disco, então a pasta `imagens/` é obrigatória no deploy.
- **Origem do bundle não documentada.** Não se sabe qual ferramenta gera o
  `index.html`. Enquanto isso, todas as edições são feitas direto no arquivo
  pelo método acima — se o bundle for algum dia regenerado pela ferramenta
  original, essas edições se perdem.

## Convenções

- Todo o conteúdo do site e as mensagens de commit são em **português**.
- Commits pequenos, um assunto por commit, no estilo do histórico existente
  (imperativo: "Troca imagem do hero…", "Adiciona silhueta do carro…").
