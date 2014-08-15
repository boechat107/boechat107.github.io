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

A primeira solução que me veio à mente foi criar um novo modelo SVM. Eu sabia que 
o modelo não estava bom, foi criado às pressas e com amostras artificiais.
Ele poderia ser melhorado usando amostras reais e mais cuidadosamente selecionadas,
por exemplo, mas isso me custaria um tempo razoável --- talvez um dia inteiro de
trabalho chato.
Daí me veio uma ideia bem interessante à cabeça, uma que me forçaria a aprender
algo novo: um corretor ortográfico. Joguei no Google.

O resultado mais interessante, na primeira página, foi um famoso 
[artigo](http://norvig.com/spell-correct.html) escrito por Peter Norvig. Artigo
excelente, mas ia além das minhas necessidades no momento. Entretanto, o termo
*edit distance* não era novo pra mim e me lembrei na hora da
[distância Levenshtein](http://en.wikipedia.org/wiki/Levenshtein_distance).
Seria simples testá-la, uma vez que existem milhares de implementações prontas
na web.

No caso de Clojure, a biblioteca [Incanter](http://incanter.org/) possui uma
implementação, que utilizei no seguinte código:

{% highlight clojure %}
(ns cetip.spell-corrector
  (:require 
    [incanter.stats :refer [levenshtein-distance]]))

(defn levenshtein-similar
  "Returns the dic's word most similar to target calculating the Levenshtein 
  distance between them. The dictionary dic is just a sequence of words."
  [target dic]
  {:pre [(string? target)]}
  (apply min-key (partial levenshtein-distance target) dic))
{% endhighlight %}

A função `levenshtein-similar` apenas calcula a distância Levenshtein entre 
a palavra a ser corrigida e cada uma do dicionário dado (sequência de palavras),
retornando aquela que tenha gerado a menor distância.

Podemos agora testar a ideia com os exemplos dados anteriormente:

{% highlight clojure %}
=> (levenshtein-similar "2509" (map str (range 1998 2015)))
2009
=> (levenshtein-similar "0012" (map str (range 1998 2015)))
2012
=> (levenshtein-similar "2097" (map str (range 1998 2015)))
2007
{% endhighlight %}

No meu caso foi fácil assumir um pequeno dicionário de datas. Usei aquele pequeno 
trecho de código no meu programa e a acurácia do OCR foi melhorada 
consideravelmente para o meu conjunto de imagens de teste.

## Conclusão

Sei que a solução implementada está longe de ser a melhor. Entretanto, a ideia 
me economizou tempo e me ajudou a abrir a cabeça para outros possíveis 
*approaches* para incrementar o meu sistema de OCR.
Talvez eu até consiga usar ideias similares para outros campos dos documentos
onde a possibilidade de palavras é pequena.
