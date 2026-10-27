# Limites territoriais do RJ: UF, RISP, AISP e CISP

Camadas oficiais do ISP-RJ com os limites da hierarquia da Blue Blood: estado (UF) → Comando de Área (RISP) → batalhão (AISP) → companhia (CISP). Servem para ligar as séries do ISPDados a um mapa e para calcular taxas por área.

| Arquivo | Conteúdo | Registros |
| --- | --- | --- |
| `uf_rj.geojson` | Contorno do estado, dissolvido das RISPs | 1 |
| `risp_rj.geojson` | Regiões Integradas de Segurança Pública | 7 |
| `aisp_rj.geojson` | Áreas Integradas de Segurança Pública, com a RISP de cada uma | 41 |
| `cisp_rj.geojson` | Circunscrições das delegacias, com a AISP e a RISP de cada uma | 137 |
| `limites_isp_rj_shp.zip` | As mesmas quatro camadas em shapefile, na resolução original | 4 camadas |
| `relacao_risp_aisp_cisp.csv` | Tabela oficial RISP × AISP × CISP, com bairros, município e região de governo | 137 |

## Fonte e preparo

- **Fonte:** [ISPDados](https://www.ispdados.rj.gov.br/), arquivos `RISPshp.rar`, `AISPshp.rar` e `CISPshp.rar` (atualizados pelo ISP em 22/10/2026) e `Relacao_RISPxAISPxCISP.csv`, baixados em 27/10/2026. Dados públicos, de download livre; citar "Fonte: Instituto de Segurança Pública (ISP-RJ)".
- **Sistema de coordenadas:** WGS 84 (EPSG:4326), longitude e latitude em graus, como no original.
- **O que mudou em relação ao ISP:**
  - Os `.rar` foram convertidos, porque o compute *Serverless* do Databricks não abre `.rar`.
  - A camada UF não existe no ISP: ela junta as 7 RISPs, e as frestas menores que 0,1 km² entre elas foram fechadas. A área bate com a soma das RISPs, 43.798 km².
  - A CISP ganhou a coluna `risp`, tirada da AISP. As 137 CISPs batem com a tabela oficial de relação, sem divergência.
  - Os GeoJSON foram simplificados (tolerância de cerca de 20 m) para ficarem leves no notebook. Para medir área ou distância com precisão, use o shapefile.
  - Colunas em minúsculas: `uf`, `cod_ibge`, `risp`, `aisp`, `cisp`, `areakm2`; na CISP, também `marcolegal` e `validacao`.

## Como levar para o Databricks

O caminho mais simples, que não depende de internet no notebook:

1. Baixe os arquivos `.geojson` desta pasta.
2. No Databricks, abra **Catalog → blue-blood → bronze → arquivos** e clique em **Upload to this volume**. Crie a pasta `geo` e envie os arquivos para ela.
3. Rode no notebook a célula SQL abaixo. Ela guarda cada camada como tabela da Bronze, com a geometria em texto GeoJSON.

```sql
USE CATALOG `blue-blood`;
CREATE OR REPLACE TABLE bronze.geo_cisp
COMMENT 'Bronze: limites das CISPs do RJ (ISPDados, 22/10/2026), geometria em GeoJSON'
AS
SELECT CAST(f.properties['cisp'] AS INT) AS cisp,
       CAST(f.properties['aisp'] AS INT) AS aisp,
       CAST(f.properties['risp'] AS INT) AS risp,
       CAST(f.properties['areakm2'] AS DOUBLE) AS area_km2,
       f.geometry AS geometria_geojson
FROM (
  SELECT explode(from_json(value,
           'features ARRAY<STRUCT<properties: MAP<STRING, STRING>, geometry: STRING>>').features) AS f
  FROM read_files('/Volumes/blue-blood/bronze/arquivos/geo/cisp_rj.geojson',
                  format => 'text', wholeText => true)
);
SELECT * FROM bronze.geo_cisp ORDER BY cisp;
```

Troque `cisp` por `aisp`, `risp` ou `uf` para as outras camadas, ajustando as colunas. Em workspaces com as funções espaciais do Databricks, `st_geomfromgeojson(geometria_geojson)` transforma o texto em geometria. Essa célula ainda não foi testada no Free Edition.

**Por que não baixar direto do ISP no notebook?** O ISP publica os limites só em `.rar`. Este repositório é privado, então o notebook também não consegue baixar daqui sem credencial. Por isso, o caminho acima usa o upload pela tela.
