# GUIA PARA O ROBÔ — app DISTRIBUIDORA

Este repositório é o app **da DISTRIBUIDORA CASAGRANDE**, um dos três apps do grupo.
Leia inteiro antes de mexer em qualquer linha.

---

## 1. O que é este app

| | |
|---|---|
| URL | https://ovoscasagrande.github.io/app-distribuidora/ |
| Repo | `Ovoscasagrande/app-distribuidora` |
| Arquivo | **`index.html` único** (526 KB) com CSS, HTML e JS dentro. Não existe `.js` nem `.css` separado |
| Backend | Supabase `ppiqkrvlvtsnsthnfibg` — **o MESMO banco dos outros 2 apps** |
| Tema | Escuro (`#121212`), cor de destaque **oliva `#797F35`** |

## 2. Regra de ouro: este app é só da distribuidora

Foi gerado a partir de um app único que atendia as 3 empresas. A separação é um bloco `window.APP`
no fim do `<script>`:

```js
window.APP = {
  "empresa": "DISTRIBUIDORA",
  "nome": "DISTRIBUIDORA CASAGRANDE",
  "telas": [ ...30 telas, incluindo as exclusivas... ]
}
```

Esse bloco faz:
1. **trava a empresa** (`empAtiva()` sempre devolve `DISTRIBUIDORA`)
2. **filtra o menu** para as telas deste app
3. **bloqueia navegação** para tela que não existe aqui
4. **trava o login**: motorista de 2 empresas (perfil `MULTI`) entra aqui travado na DISTRIBUIDORA

**Não remova nem edite esse bloco.**

## 3. As telas EXCLUSIVAS deste app

Estas só existem aqui — é o coração da distribuidora:

| Tela | O que faz |
|---|---|
| `pedidos-rota` | lê os PDFs de pedido do ERP, casa/atualiza cliente por CNPJ, monta a rota e gera o nº do roteiro |
| `roteiros` | lista os roteiros: aceitar, iniciar, finalizar, cancelar, baixa manual |
| `clientes` | `clientes_dist` — 179 clientes cadastrados |
| `produtos` | catálogo da distribuidora (12 SKUs) — **separado do catálogo da OVOS** |
| `rotas` | a rota do dia com ordem de entrega |
| `carga` | carregamento da distribuidora |
| `nfs` | notas fiscais em PDF |

**Cuidado especial com o parser de PDF** (`parsePedido`, `parseDanfe`, `detectTipoDoc`, `geraPdfCarga`,
`geraPdfCargaCliente`, `geraPdfRoteiro`). Ele lê a descrição do item que **quebra em duas linhas**
no PDF (`"Manteiga Cremosa Com Sal Teixeira"` / `"12x500g Pote"`). Já houve bug em que 12x500g e
24x200g saíam iguais. Se mexer, teste com os PDFs reais.

## 4. O que você pode fazer

- Melhorar as telas acima, layout, textos, atalhos
- Criar telas novas da distribuidora (acrescente o nome em `window.APP.telas`)
- Melhorar o parser dos PDFs do ERP, a sequência de entrega, os comprovantes
- Relatórios de entrega, km por rota, custo por entrega
- **Pedágios:** a tabela `pedagios` já tem os lançamentos do Sem Parar (16 placas, 669 registros).
  Dá para cruzar custo de pedágio por rota.

## 5. O que você NÃO pode fazer

| Proibido | Por quê |
|---|---|
| Renomear ou remover `id="..."` | O JS busca por `getElementById`. Hoje são **390 IDs** |
| Renomear tabelas/colunas do Supabase | O banco é compartilhado com os outros 2 apps |
| Renomear RPCs | Mesmo motivo |
| Mexer em `usuarios`, `motoristas`, `veiculos` | **Base compartilhada do grupo** — ver seção 7 |
| Misturar produtos da OVOS com os da distribuidora | Códigos iguais são produtos **diferentes**. O código `103` é só da OVOS |
| Usar `alert()` / `confirm()` / `prompt()` | Use `toast()` e o modal `#modal-wrap` |
| Emoji dentro de `doc.text()` / `toast()` | jsPDF vira ■ e toast é texto puro |
| Mexer no `sw.js` sem subir a versão | O cache do celular impede a atualização |

## 6. Regras de produto (não negociáveis)

- Datas **DD-MM-AAAA**.
- Valores pt-BR com vírgula. Use `parseBR()` / `numQtd()`; **nunca** `parseFloat` em campo digitado.
  `6.245` em quantidade = **6,245** (decimal), não 6245. `21.500` em km = 21500 (milhar).
- Peso decimal no PDF: `6.31` não pode virar `631`.
- Alvos de toque ≥ 48 px.

## 7. O BANCO É COMPARTILHADO

### Grupo A — base do grupo (os 3 apps usam os mesmos registros)
`usuarios` · `motoristas` · `veiculos` · `motorista_veiculo` · `veiculos_documentos` · `postos`
O motorista é a mesma pessoa nos 3 apps (coluna `empresas` jsonb).
**Alterar a estrutura dessas tabelas quebra os outros dois apps.**

### Grupo B — operação (têm coluna `empresa`)
`jornadas` · `abastecimentos` · `despesas` · `recebimentos` · `avisos` · `multas` ·
`ordens_servico` · `auditoria` · `tipos_despesa` · `tipos_recebimento`
O app filtra por `empFilter()` (aceita `AMBAS` e `TODAS`).

### Grupo C — exclusivas daqui (sem coluna empresa)
`clientes_dist` · `rotas` · `entregas` · `notas_fiscais` · `nf_itens` · `pedidos`

### Exclusivas de outros apps (não use)
- **OVOS:** `clientes_promotoria`, `roteiros_promotoria`, `visitas`
- **TRANSPORTADORA:** nenhuma tabela própria

## 8. Se precisar de mudança no banco

Você **não** roda SQL. Descreva a mudança e o **CASA** roda, avisando o dono.
A exceção é o grupo C (exclusivo daqui) — ainda assim, peça antes.

## 9. Antes de publicar

1. `node validar_integracao.js NOVO.html ORIGINAL.html` → **APROVADO**
2. Testar em 390 px
3. Subir a versão do cache no `sw.js` (`cg-dist-v1` → `cg-dist-v2`)
4. Testar logado como **gerência** e como **motorista**
5. Gerar um relatório de carga de verdade e conferir descrição + quantidade

## 10. Perfis que este app atende

`gerencia` · `motorista` · `conferente` · `mecanico` · `pendente` · `vendedor`
(`promotor` é da OVOS — não existe aqui.)

---

## Dúvida? Pergunte antes de inventar.

Uma linha errada derruba a tela de um motorista em produção.
