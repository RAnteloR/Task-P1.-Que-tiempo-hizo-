# P1 · Que tempo fixo

Análise dos datos diarios da estación de A Coruña / Alvedro (aeroporto, indicativo 08002), do 1 de xaneiro de 2021 ao 31 de decembro de 2025.

## Estrutura

```
├── README.md
├── requirements.txt
├── .gitignore
├── datos/
│   ├── raw/     # ficheiro orixinal, tal como chega
│   └── clean/   # datos tratados
├── notebooks/
└── output/
```

## Datos

- **Fonte:** [Meteostat](https://meteostat.net/) (o ficheiro simula a exportación dun portal de datos e foi modificado para incluír problemas de calidade).
- **URL de descarga:** <https://raw.githubusercontent.com/bigdatawirtz/datasets/main/datasets/meteo_alvedro_2021_2025.csv>
- **Variables do proxecto:** temperatura media, mínima e máxima e precipitación. Tamén inclúe humidade, vento e presión.
- O ficheiro gárdase sen modificar en `datos/raw/meteo_alvedro_2021_2025.csv`.

## Uso

```bash
pip install -r requirements.txt
jupyter notebook notebooks/Task1.ipynb
```

O notebook pódese executar de principio a fin desde un cartafol `datos/` baleiro: crea os cartafoles necesarios e descarga o ficheiro.
