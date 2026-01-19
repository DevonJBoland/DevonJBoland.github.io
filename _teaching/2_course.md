---
layout: page
title: Introduction to AlphaFold3
description: 8th Annual Texas A&M Research Computing Symposium Workshop - Co-led with Dr. Michael Dickens 
img: assets/img/teaching/AF_workshop.jpg
importance: 2
category: current
---

## Materials Used in HPRC Hosted Workshop

Example JSON file for prediction of a Homodimeric protein
```json
{
  "name": "CSAS",
  "modelSeeds": [1, 2],
  "sequences": [
    {
      "protein": {
        "id": ["A", "B"],
        "sequence": "MIKLKPATFLILLVFLFETGCTENCLSNDIHALILARGGSKGIKLKNLAEIGGSSLLARTIMTIKNSTCFRHIWVSTDDKRIAIEAQKYGAIIHHRPEKFARDDTPSLHAISEFLDVHRSIHDFALFQCTSVFLKTKYIQEAVRKFESHDCVFAAKRSHYLRWKVVDGELMPAEFDLSARPRRQDWQGDIVETGMFYFSRRKLVDSGLLQNNRCSVVEIDAKDSLEIDSSHDLTLAKYILSSETKTEL"
       }
    }
  ],
  "dialect": "alphafold3",
  "version": 2
}
```

Example JSON file for predictiong of a monomeric protein with ligands and glycosulation
```json

```