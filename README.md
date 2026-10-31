# Dados do app Genérico

Preços máximos ao consumidor (PMC) de medicamentos, preparados a partir da lista oficial da
**CMED/Anvisa** para uso offline no app Genérico.

- **Fonte:** Câmara de Regulação do Mercado de Medicamentos (CMED), Agência Nacional de
  Vigilância Sanitária (Anvisa), "Lista de preços de medicamentos – Preço Máximo ao Consumidor",
  publicada mensalmente em
  <https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/cmed/precos>.
- Os dados não foram alterados: apenas reorganizados (EANs inválidos removidos, colunas de PMC
  convertidas em centavos, apresentações agrupadas por princípio ativo, concentração e forma).
- Medicamentos de uso restrito a hospitais não têm PMC de varejo e aparecem sem preço.
- As alíquotas de ICMS por estado (`icms_states.json`) vêm de fontes secundárias, citadas no
  próprio arquivo. Só os estados marcados como `verified` foram conferidos na legislação.

Este repositório não dá orientação médica. Para trocar de medicamento, fale com o farmacêutico
ou com quem prescreveu.

## O que tem aqui

| Arquivo | Conteúdo |
|---|---|
| `manifest.json` | versão atual: mês dos dados, arquivos, tamanhos e sha256 |
| `manifest.json.sig` | assinatura Ed25519 de `manifest.json` (base64) |
| `signing_keys.json` | chaves públicas usadas nas assinaturas |
| `icms_states.json` | alíquota de ICMS por estado, com fontes |
| `diffs/` | mudanças linha a linha em relação ao mês anterior |
| `reports/` | relatório de cada atualização (verificações e maiores variações) |
| `CHANGELOG.md` | histórico de atualizações |

O banco completo (`generico-AAAA-MM.sqlite.gz`) e os diffs compactados ficam nas
[Releases](../../releases).

## Como conferir a assinatura

```sh
pip install cryptography
python3 - <<'EOF'
import base64, json
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey
manifest = open("manifest.json", "rb").read()
sig = base64.b64decode(open("manifest.json.sig").read())
key_id = json.loads(manifest)["key_id"]
key = next(k for k in json.load(open("signing_keys.json"))["keys"] if k["key_id"] == key_id)
Ed25519PublicKey.from_public_bytes(base64.b64decode(key["public_key"])).verify(sig, manifest)
print("assinatura válida:", key_id)
EOF
```

Depois confira o sha256 de cada arquivo baixado com o valor em `manifest.json`.

## Como as atualizações acontecem

Um processo automático verifica todo dia se a CMED publicou uma lista nova. Se houver, ele
monta o banco, faz verificações de consistência e abre um pull request aqui com o relatório.
A atualização só é publicada depois que o pull request é aprovado. Se alguma verificação
falhar, nada é publicado e uma issue é aberta.
