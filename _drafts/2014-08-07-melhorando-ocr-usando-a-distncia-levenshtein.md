---
layout: post
title: "Melhorando OCR usando a distância Levenshtein"
description: "Aplicando a distância Levenshtein para melhorar acurácia de OCR"
category: programming
tags: [Image Processing, OCR, Spell Checking]
mathjax: 
lang: pt-br
trans: /increasing-ocrs-accuracy-with-levenshtein/
---
{% include JB/setup %}

*Estreando a primeira tradução...*

## Problema

Quase um mês atrás, estava eu fazendo algumas melhorias no meu programa de OCR e
pensando sobre alguns caracteres classificados incorretamente que estavam 
aparecendo na tela.
O programa deveria processar fotografias de alguns documentos específicos compostos
por diversos campos, onde um deles contem um número representando um ano, uma data.
Fiquei frustrado porque muitas vezes o número reconhecido continha um ou dois
dígitos incorretamente classificados, mas que eu conseguia facilmente chutar o 
número correto.
Por exemplo:

* 2509
* 0012
* 2097

OK, chutar não necessariamente me levaria ao número correto --- os últimos dois 
dígitos poderiam estar errados --- mas provavelmente seria uma solução melhor 
que a do meu OCR.

## A ideia da solução


