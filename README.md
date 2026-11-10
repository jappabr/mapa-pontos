# mapa-pontos

Pontos de iluminação pública por município, exportados do SIIP (consulta SQL 217 —
"INVENTARIO COM LAMPADAS") e atualizados pelo script `puxar_pontos.py`.

## Arquivos

- `municipios/index.json` — lista dos municípios, com o caminho do arquivo, total de pontos e data da última atualização.
- `municipios/<id>-<nome>.geojson` — um `FeatureCollection` por município. Cada feature é um poste (`Point` em `[lon, lat]`) com:
  - `id`, `numero_poste`, `endereco`, `bairro`
  - `lampadas`: lista de `{produto, tipo, potencia, posicao}`
  - `qtd_lampadas`, `potencia_total`

## Como usar no site

```js
const BASE = "https://raw.githubusercontent.com/jappabr/mapa-pontos/main/";

const indice = await fetch(BASE + "municipios/index.json", { cache: "no-store" }).then(r => r.json());
const cotia = indice.municipios.find(m => m.id === 2633);
const pontos = await fetch(BASE + cotia.arquivo, { cache: "no-store" }).then(r => r.json());

// Leaflet
L.geoJSON(pontos, {
  pointToLayer: (f, latlng) => L.circleMarker(latlng, { radius: 4 }),
  onEachFeature: (f, layer) => layer.bindPopup(
    `<b>Poste ${f.properties.numero_poste}</b><br>${f.properties.endereco}<br>` +
    f.properties.lampadas.map(l => `${l.tipo} ${l.potencia ?? ""}W`).join("<br>")
  ),
}).addTo(map);
```

O `raw.githubusercontent.com` libera CORS e tem cache de ~5 minutos, então uma atualização
publicada aparece nos sites em poucos minutos.
