---
layout: post
title:  "Objeto hashable em Python"
date:   2022-07-24 20:38:37 -0300
author: itepifanio
categories: [ Python, Data Structure ]
comments: false
---

Hash de objetos é uma representação númerica inteira que é obtida utilizando o [dunder method](https://docs.python.org/3/reference/datamodel.html#special-method-names) `__hash__`. Compreender esse conceito ajuda a entender como as estruturas de dados Python funcionam, uma vez que o hash dos objetos são utilizados internamente. 

Programadores Python costumam se deparar com o conceito de hash de objetos quando tentam armazenar um objeto sem hash em uma estrutura de dados da linguagem. Por exemplo:

```python
from dataclasses import dataclass

@dataclass
class Pessoa:
    nome: str
    cpf: str

p = Pessoa(nome="Ítalo Epifânio", cpf="1010101010")
pessoas = set()
pessoas.add(p)
```

O código acima define uma classe pessoa e um objeto `p` do tipo pessoa. Ao tentar adicionar uma pessoa ao conjunto `pessoas` o seguinte erro é lançado:

```shell
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'Pessoa'
```

Isso acontece porque a estrutura de dados "conjunto" do Python utiliza o hash em suas tabelas internas para encontrar o valor do objeto rapidamente. Como nosso objeto não tem hash, o erro acima é lançado.

Para adicionar um hash a um objeto implementa-se as funções `__hash__` e `__eq__` simultaneamente (caso esteja utilizando Python 2 a função `__ne__` também deve ser adicionada). No exemplo a seguir modificamos nossa classe anterior para adquirir essa propriedade.

```python
@dataclass
class Pessoa:
    nome: str
    cpf: str

    def __hash__(self):
        return hash(self.cpf)
    
    def __eq__(self, other):
        mesma_classe = self.__class__ == other.__class__
        mesmo_cpf = self.cpf == other.cpf
        
        return mesma_classe and mesmo_cpf

p = Pessoa(nome="Ítalo Epifânio", cpf="101.010.101-01")
pessoas = set()
pessoas.add(p)
```

Agora o código consegue ser executado sem o erro anterior. Se executarmos `print(pessoas)` veremos que o conjunto contém o seguinte valor:

```shell
{Pessoa(nome='Ítalo Epifânio', cpf=1010101010)}
```

> Um objeto é dito hashable se o valor de hash nunca é modificado durante sua fase de vida
> -- <cite> [Python Docs](https://docs.python.org/3/glossary.html#term-hashable) </cite>

Apenas implementar os dunder métodos acima não garante que o objeto é hashable. É preciso garantir o valor de hash desse objeto jamais sejá alterado pois isso pode levar a comportamentos não esperados, por exemplo:

```python 
p = Pessoa(nome="Ítalo Epifânio", cpf="101.010.101-01")
pessoas = set()
pessoas.add(p)
print(p in pessoas) # retorna True
p.cpf = "999.999.999-99"
print(p in pessoas) # retorna False
print(pessoas) # retorna {Pessoa(nome='Ítalo Epifânio', cpf='999.999.999-99')}
```

Ao alterar o cpf do objeto `p` note que o objeto não é mais encontrado na estrutura de dados conjunto (a segunda chamada do `print(p in pessoas)` retorna falso) e quando lista-se os valores do conjunto `pessoas` nota-se que há ainda um valor lá.

Em resumo: você não pode basear hash de objetos em valores mutáveis. Se o atributo de um objeto pode ser modificado durante seu ciclo de vida comportamentos inesperados podem acontecer.