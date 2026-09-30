# Mapa Astra

Mapa territorial interativo desenvolvido para uso em dashboards do projeto Astra, com foco em recortes por **estado** e **município** e enquadramento automático da região selecionada.

A aplicação é estática, roda inteiramente no navegador e pode ser publicada pelo **GitHub Pages**. O mapa usa **Leaflet** para renderização e carrega os limites municipais a partir do repositório público `tbrugz/geodata-br`.

## Objetivo

O objetivo principal é contornar uma limitação comum dos mapas de região do Metabase: quando um filtro é aplicado, o Metabase normalmente destaca a geometria correspondente, mas mantém a extensão espacial do mapa inteiro.

Este projeto faz diferente:

- se um estado é informado, exibe apenas os municípios daquela UF;
- se um município é informado, exibe apenas aquele município;
- ajusta automaticamente o enquadramento com `fitBounds()`;
- permite controlar o mapa por parâmetros na URL;
- pode ser incorporado em dashboards via `iframe` ou outro mecanismo de embedding.

## Como funciona

A página lê os parâmetros da URL:

```text
?estado=Pará
```

ou:

```text
?estado=Pará&municipio=Belém
```

A partir desses parâmetros, o JavaScript:

1. identifica a UF correspondente;
2. carrega o GeoJSON municipal daquela UF;
3. filtra o município, quando informado;
4. renderiza apenas as feições selecionadas;
5. aplica `map.fitBounds()` para ajustar automaticamente o zoom.

Fluxo resumido:

```text
Filtro externo / URL
        ↓
estado + município
        ↓
GeoJSON da UF
        ↓
filtragem das feições
        ↓
Leaflet
        ↓
fitBounds()
        ↓
mapa enquadrado na região selecionada
```

## Exemplos de uso

### Exibir todos os municípios de um estado

```text
https://cleytoncandeira.github.io/mapa-astra/?estado=Pará
```

Resultado esperado: todos os municípios do Pará, com o mapa enquadrado na extensão territorial do estado.

### Exibir apenas um município

```text
https://cleytoncandeira.github.io/mapa-astra/?estado=Pará&municipio=Belém
```

Resultado esperado: apenas o polígono de Belém, com zoom automático na geometria.

### Usar sigla da UF

O mapa também aceita a sigla da unidade federativa:

```text
https://cleytoncandeira.github.io/mapa-astra/?estado=PA&municipio=Belém
```

## Parâmetros disponíveis

| Parâmetro | Obrigatório | Exemplo | Descrição |
|---|---:|---|---|
| `estado` | Sim | `Pará` ou `PA` | Define a unidade federativa a ser carregada. |
| `municipio` | Não | `Belém` | Filtra um município específico dentro da UF. |

Os nomes são normalizados internamente, portanto acentos e diferenças simples de caixa não impedem a busca. Por exemplo, `Pará`, `para` e `PARÁ` são tratados de forma equivalente.

## Fonte geográfica

Os limites municipais são carregados a partir do projeto:

```text
https://github.com/tbrugz/geodata-br
```

Cada UF possui um arquivo GeoJSON próprio. O Pará, por exemplo, é carregado a partir de:

```text
geojson/geojs-15-mun.json
```

As propriedades utilizadas atualmente são:

```json
{
  "id": "1501402",
  "name": "Belém",
  "description": "Belém"
}
```

O campo `id` corresponde ao código IBGE do município e `name` é utilizado na filtragem pelo parâmetro `municipio`.

## Tecnologias

- HTML5
- JavaScript
- Leaflet 1.9.4
- OpenStreetMap como mapa-base
- GeoJSON
- GitHub Pages

Não há necessidade de backend, banco de dados, Node.js, Python ou processo de build para executar a versão atual.

## Estrutura do repositório

Atualmente o projeto é propositalmente simples:

```text
mapa-astra/
├── index.html
└── README.md
```

Toda a lógica do mapa está concentrada em `index.html`.

Isso facilita a publicação no GitHub Pages e reduz dependências, mas a estrutura pode ser separada futuramente em arquivos como:

```text
mapa-astra/
├── index.html
├── css/
│   └── mapa.css
├── js/
│   └── mapa.js
└── README.md
```

## Executar localmente

Como a página usa `fetch()` para carregar arquivos GeoJSON remotos, o ideal é servir o diretório com um servidor HTTP local.

Com Python:

```bash
python -m http.server 8000
```

Depois abra:

```text
http://localhost:8000/?estado=Pará&municipio=Belém
```

Também é possível usar extensões como Live Server no VS Code.

## Publicação no GitHub Pages

No repositório do GitHub:

1. abra **Settings**;
2. acesse **Pages**;
3. em **Build and deployment**, selecione `Deploy from a branch`;
4. escolha a branch `main`;
5. selecione a pasta `/(root)`;
6. salve.

Após o deploy, a aplicação fica disponível em:

```text
https://cleytoncandeira.github.io/mapa-astra/
```

## Integração com Metabase

A ideia é usar o mapa como uma visualização externa controlada por parâmetros.

Exemplo conceitual:

```html
<iframe
  src="https://cleytoncandeira.github.io/mapa-astra/?estado=Pará&municipio=Belém"
  width="100%"
  height="600"
  frameborder="0">
</iframe>
```

O objetivo final é fazer os filtros do dashboard alimentarem dinamicamente os parâmetros:

```text
estado
municipio
```

para produzir URLs como:

```text
?estado=Pará
```

ou:

```text
?estado=Pará&municipio=Belém
```

### Comportamento desejado no dashboard

| Seleção no dashboard | Resultado no mapa |
|---|---|
| Nenhuma seleção | comportamento padrão a definir |
| Estado = Pará | todos os municípios do Pará |
| Estado = Pará + Município = Belém | apenas Belém |

A integração exata dependerá da forma de embedding disponível na instalação do Metabase.

## Enquadramento automático

O comportamento central do projeto é executado por:

```javascript
map.fitBounds(layer.getBounds(), {
  padding: [20, 20],
  maxZoom: municipio ? 11 : 8
});
```

Isso faz com que a extensão do mapa seja calculada a partir das geometrias realmente exibidas.

Assim, o mapa não apenas muda a cor ou o destaque de uma região: ele efetivamente enquadra a seleção territorial.

## Normalização de nomes

Para evitar falhas causadas por acentos ou capitalização, o projeto normaliza nomes antes da comparação.

Exemplo:

```javascript
function normalizar(txt) {
  return (txt || '')
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .trim()
    .toLowerCase();
}
```

Isso permite comparar corretamente valores como:

```text
Belém
belem
BELÉM
```

## Códigos das UFs

Os arquivos do `geodata-br` utilizam o código IBGE da unidade federativa no nome do arquivo.

Alguns exemplos:

| UF | Código IBGE | Arquivo |
|---|---:|---|
| PA | 15 | `geojs-15-mun.json` |
| SP | 35 | `geojs-35-mun.json` |
| MG | 31 | `geojs-31-mun.json` |
| RJ | 33 | `geojs-33-mun.json` |
| AM | 13 | `geojs-13-mun.json` |

O `index.html` contém um dicionário com todas as UFs brasileiras para converter nome ou sigla no código correspondente.

## Limitações atuais

A versão inicial ainda possui algumas limitações:

- depende da disponibilidade do GeoJSON remoto;
- não há cache local das geometrias;
- o estado é obrigatório na versão atual;
- não existe ainda sincronização automática com filtros do Metabase;
- o estilo cartográfico ainda é genérico;
- não há legenda ou indicadores temáticos;
- o mapa usa dados municipais separados por UF, e não uma camada nacional única;
- ainda não existe tratamento especial para municípios homônimos entre estados, embora o carregamento prévio por UF já reduza esse problema.

## Próximas melhorias

Evoluções previstas para o projeto:

- integração dinâmica com filtros do Metabase;
- modo nacional quando nenhum estado estiver selecionado;
- suporte a polígonos estaduais;
- identidade visual do Astra;
- cores e tipografia customizadas;
- remoção ou substituição do mapa-base, quando desejável;
- tooltip personalizado;
- legenda dinâmica;
- suporte a indicadores quantitativos por município;
- classificação por faixas de valores;
- carregamento de dados de uma API ou consulta externa;
- possibilidade de receber código IBGE diretamente pela URL;
- armazenamento local dos GeoJSONs para reduzir dependência externa.

## Possível evolução para indicadores

Além do simples recorte territorial, a camada pode futuramente receber valores quantitativos.

Exemplo conceitual:

```text
id_municipio | nome       | valor
1501402      | Belém      | 87.3
1500800      | Ananindeua | 62.1
```

A partir disso, o mapa pode aplicar uma função de estilo baseada em `valor` e funcionar como um choropleth completo.

Isso permitiria combinar:

```text
filtros do Metabase
        +
indicadores do BigQuery
        +
polígonos municipais
        ↓
mapa territorial interativo
```

## Segurança e privacidade

A aplicação é estática e não possui autenticação própria. Os parâmetros presentes na URL são públicos para quem acessa o endereço.

Por isso, não devem ser enviados pela URL dados sensíveis, credenciais ou informações privadas.

## Desenvolvimento

Repositório:

```text
https://github.com/cleytoncandeira/mapa-astra
```

Branch principal:

```text
main
```

Arquivo principal:

```text
index.html
```

## Licenciamento e atribuições

Este projeto utiliza bibliotecas e dados de terceiros. Antes de uso em produção ou redistribuição, é recomendável conferir e respeitar as licenças e exigências de atribuição de cada fonte, especialmente:

- Leaflet;
- OpenStreetMap;
- `tbrugz/geodata-br`;
- demais fontes de dados que venham a ser adicionadas ao projeto.

A atribuição ao OpenStreetMap já é exibida no mapa-base pela configuração do Leaflet.
