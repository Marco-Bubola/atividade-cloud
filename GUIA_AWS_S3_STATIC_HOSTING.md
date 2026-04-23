# Guia Técnico — AWS S3 Static Website Hosting
## Desafio C: Hospedagem de Site Estático

---

## Pré-requisitos
- Conta ativa na AWS (https://aws.amazon.com/free)
- Arquivo `index.html` pronto para upload
- Acesso ao AWS Management Console

---

## PARTE 1 — Criação do Bucket S3

### Passo 1: Acessar o Amazon S3
1. Faça login no [AWS Management Console](https://console.aws.amazon.com)
2. Na barra de pesquisa superior, digite **S3** e clique no serviço
3. Clique no botão **"Create bucket"** (laranja, canto superior direito)

### Passo 2: Configurar o Bucket
| Campo | Valor |
|---|---|
| **Bucket name** | `patafeliz-[suas-iniciais]-2026` (deve ser único globalmente) |
| **AWS Region** | `us-east-1` (N. Virginia) — mais barata |
| **Object Ownership** | ACLs disabled (padrão) |

> **Dica de nome único:** use `patafeliz-fatec-[rm]-2026` substituindo [rm] pelo seu RM.

### Passo 3: Configurar Acesso Público (CRÍTICO)
No bloco **"Block Public Access settings for this bucket"**:
- **DESMARQUE** a opção "Block all public access"
- Uma caixa de confirmação aparecerá — marque a caixa e confirme

> ⚠️ Normalmente mantemos o bloqueio ativado por segurança, mas para hosting público precisamos desativá-lo.

### Passo 4: Finalizar Criação
- Deixe as demais opções como padrão (Versioning: Disabled)
- Role até o final e clique em **"Create bucket"**
- O bucket aparecerá na lista ✓

---

## PARTE 2 — Ativar Static Website Hosting

### Passo 5: Acessar as Propriedades do Bucket
1. Clique no nome do bucket recém-criado
2. Clique na aba **"Properties"** (última aba)
3. Role até o final até encontrar **"Static website hosting"**
4. Clique em **"Edit"**

### Passo 6: Configurar o Hosting
| Campo | Valor |
|---|---|
| **Static website hosting** | Enable |
| **Hosting type** | Host a static website |
| **Index document** | `index.html` |
| **Error document** | `error.html` (opcional) |

5. Clique em **"Save changes"**
6. Volte à aba Properties → Static website hosting → **copie a URL do Bucket website endpoint**
   - Formato: `http://[bucket-name].s3-website-us-east-1.amazonaws.com`

> **Esta é a URL que você entregará** — não a URL direta do arquivo.

---

## PARTE 3 — Upload do Arquivo index.html

### Passo 7: Fazer Upload
1. Clique na aba **"Objects"** do bucket
2. Clique em **"Upload"**
3. Clique em **"Add files"** e selecione o `index.html`
4. Clique em **"Upload"** (não precisa mudar nada)
5. Aguarde a confirmação de upload ✓

---

## PARTE 4 — Configurar Política de Acesso Público (Bucket Policy)

Sem essa etapa o site retorna **403 Forbidden**.

### Passo 8: Adicionar Bucket Policy
1. Clique na aba **"Permissions"**
2. Role até **"Bucket policy"** e clique em **"Edit"**
3. Cole o JSON abaixo (substitua `SEU-BUCKET-NAME` pelo nome real):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::SEU-BUCKET-NAME/*"
    }
  ]
}
```

4. Clique em **"Save changes"**
5. O bucket mostrará o badge **"Publicly accessible"** em vermelho — isso é esperado ✓

---

## PARTE 5 — Teste Final

### Passo 9: Verificar o Site
1. Volte à aba **"Properties"**
2. Role até **"Static website hosting"**
3. Clique na URL do endpoint
4. O site `index.html` deve carregar corretamente no navegador ✓

**URL entregável:** `http://patafeliz-fatec-[rm]-2026.s3-website-us-east-1.amazonaws.com`

---

## Estimativa de Custo — 1 TB de Dados

| Item | Valor (us-east-1) |
|---|---|
| Armazenamento (1 TB/mês) | $23,00 |
| Requisições PUT/COPY/POST (por 1.000) | $0,005 |
| Requisições GET (por 1.000) | $0,0004 |
| Transferência de dados saindo (primeiro 1 GB) | Gratuito |
| Transferência de dados saindo (TB adicional) | $90,00/TB |
| **Total estimado (só armazenamento)** | **~$23/mês** |

> **Comparativo:** Azure Blob Storage LRS ≈ $20,80/TB/mês | Google Cloud Storage Standard ≈ $23/TB/mês  
> Os três provedores têm preços muito similares para armazenamento padrão.

---

## Possíveis Erros e Soluções

| Erro | Causa | Solução |
|---|---|---|
| 403 Forbidden | Bucket policy não aplicada | Refaça o Passo 8 |
| 403 Forbidden | Block Public Access ainda ativo | Verifique o Passo 3 |
| 404 Not Found | Nome do arquivo errado | Confirme que o arquivo se chama exatamente `index.html` |
| URL não abre | Usando URL errada | Use a URL de **endpoint do website**, não a URL do objeto |
| Bucket name inválido | Nome com maiúsculas ou caracteres especiais | Use apenas letras minúsculas, números e hífens |

---

## Análise Crítica (para o Relatório)

### Pontos Positivos da AWS S3
- Console bem documentado e com tutoriais embutidos
- Processo de hosting estático é intuitivo após conhecer o fluxo
- Preço competitivo e Free Tier generoso
- Integração nativa com CloudFront (CDN) para HTTPS

### Pontos de Atenção
- **Block Public Access** pode confundir iniciantes — a AWS prioriza segurança por padrão
- A Bucket Policy em JSON pode assustar quem não tem familiaridade com IAM
- URL final usa HTTP (não HTTPS) — para HTTPS é necessário configurar CloudFront
- Não há "modo fácil" para hosting estático como no Netlify/Vercel

### Tempo estimado de configuração
- Com este guia: **5 a 10 minutos**
- Sem guia (explorando o console): **20 a 40 minutos**
