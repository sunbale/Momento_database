# Momento_database

# Nível 1: Conhecendo a Empresa
Seu Primeiro Dia na Momento
Você acabou de ser contratado pela Momento! Vamos começar incluindo você no sistema.

1.1 Inclua suas próprias informações no departamento de Tecnologia da empresa.

R:

db.funcionarios.insertOne({
  "nome": "Giovana",
  "sobrenome": "Sunbale",
  "data_nascimento": "2003-07-03",
  "data_contratacao": "2026-11-19",
  "salario": 5000,
  "departamento": "Tecnologia",
  "cargo": "Desenvolvedora",
  "escritorio": "São Paulo"
})
1.2 Agora que você faz parte da equipe, quantos funcionários temos ao total na empresa?
R: 50 funcionários
db.funcionarios.countDocuments()

1.3 Quantos funcionários trabalham especificamente no Departamento de Tecnologia?
R: 8 funcinários

db.funcionarios.countDocuments({
  departamento_id: 6

1.4 Liste todos os departamentos que existem na empresa. Quantos são?
R:'Administração, 'Marketing', Suprimentos, Recursos Humanos, Transportes, Tecnologia, Relações Públicas, Vendas, Executivo, Finanças, Contabilidade, Biotecnologia e Tecnologias Avançadas.

db.departamentos.find().pretty()

1.5 Quantos escritórios a Momento possui? Em quais países?
R: 9, 'CA', 'DE', 'UK', 'US', 'WA'

db.escritorio.countDocuments()
db.escritorio.distinct("pais_id")


# Nível 2: Análise Financeira Básica
O CFO precisa de relatórios urgentes sobre os custos da empresa.

2.1 Quantos funcionários trabalham no Departamento de Vendas?
R: 5
db.funcionarios.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "dados_departamento"
    }
  },
  {
    $unwind: "$dados_departamento"
  },
  {
    $match: {
      "dados_departamento.nome": "Vendas"
    }
  },
  {
    $count: "total_funcionarios"
  }
])

2.2 Qual é o custo total com salários do Departamento de Vendas?
R: custo_total_salarios: Decimal128('51500.00')

db.funcionarios.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "dados_departamento"
    }
  },
  {
    $unwind: "$dados_departamento"
  },
  {
    $match: {
      "dados_departamento.nome": "Vendas"
    }
  },
  {
    $group: {
      _id: null,
      custo_total_salarios: {
        $sum: "$salario"
      }
    }
  }
])

2.3 Qual é a média salarial da empresa, excluindo os cargos de CEO, CMO e CFO?
R:   media_salarial: Decimal128('9355.000217391304347826086956521739')

db.funcionarios.aggregate([
  {
    $lookup: {
      from: "cargos",
      localField: "cargo_id",
      foreignField: "_id",
      as: "dados_cargo"
    }
  },
  {
    $unwind: "$dados_cargo"
  },
  {
    $match: {
      "dados_cargo.nome": {
        $nin: ["CEO", "CMO", "CFO"]
      }
    }
  },
  {
    $group: {
      _id: null,
      media_salarial: {
        $avg: "$salario"
      }
    }
  }
])

2.4 Qual é a média salarial do Departamento de Tecnologia?
R:   media_salarial: Decimal128('6787.50')
db.funcionarios.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "dados_departamento"
    }
  },
  {
    $unwind: "$dados_departamento"
  },
  {
    $match: {
      "dados_departamento.nome": "Tecnologia"
    }
  },
  {
    $group: {
      _id: "$dados_departamento.nome",
      media_salarial: {
        $avg: "$salario"
      }
    }
  }
])

2.5 Qual departamento possui a maior média salarial?
R:  _id: 'Tecnologias Avançadas',
  media_salarial: Decimal128('21657.50')

  db.funcionarios.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "dados_departamento"
    }
  },
  {
    $unwind: "$dados_departamento"
  },
  {
    $group: {
      _id: "$dados_departamento.nome",
      media_salarial: {
        $avg: "$salario"
      }
    }
  },
  {
    $sort: {
      media_salarial: -1
    }
  },
  {
    $limit: 1
  }
])

2.6 Qual departamento possui o menor número de funcionários?
R:  _id: 1,
  nome: 'Administração',
  quantidade_funcionarios: 1

  db.departamentos.aggregate([
  {
    $lookup: {
      from: "funcionarios",
      localField: "_id",
      foreignField: "departamento_id",
      as: "funcionarios_departamento"
    }
  },
  {
    $project: {
      nome: 1,
      quantidade_funcionarios: {
        $size: "$funcionarios_departamento"
      }
    }
  },
  {
    $sort: {
      quantidade_funcionarios: 1
    }
  },
  {
    $limit: 1
  }
])

# Nível 3: Recursos Humanos
O RH está fazendo uma análise demográfica da empresa.

3.1 Quantos funcionários da empresa Momento possuem cônjuges?
R: 4

db.funcionarios.countDocuments({
  dependentes: {
    $elemMatch: {
      relacionamento: "Cônjuge"
    }
  }
})

3.2 Quantos funcionários possuem filhos registrados?
R: 30

db.funcionarios.countDocuments({
  dependentes: {
    $elemMatch: {
      relacionamento: "Filha(o)"
    }
  }
})

3.3 Qual funcionário foi contratado há mais tempo na empresa?
R: primeiro_nome: 'Madeleine',
  sobrenome: 'Prior',
  data_contratacao: ISODate('1987-06-17T00:00:00.000Z')
  
  db.funcionarios.find(
  {},
  {
    primeiro_nome: 1,
    sobrenome: 1,
    data_contratacao: 1,
    _id: 0
  }
).sort({
  data_contratacao: 1
}).limit(1)

3.4 Qual funcionário foi contratado há menos tempo na empresa?
R:  primeiro_nome: 'Samus',
  sobrenome: 'Aran',
  data_contratacao: ISODate('2024-11-08T00:00:00.000Z')

db.funcionarios.find(
  {},
  {
    primeiro_nome: 1,
    sobrenome: 1,
    data_contratacao: 1,
    _id: 0
  }
).sort({
  data_contratacao: -1
}).limit(1)

3.5 Liste os 5 funcionários com mais tempo de casa, ordenados pela data de contratação.
R: primeiro_nome: 'Madeleine',
  sobrenome: 'Prior',
  data_contratacao: ISODate('1987-06-17T00:00:00.000Z')
   primeiro_nome: 'Jennifer',
  sobrenome: 'Whalen',
  data_contratacao: ISODate('1987-09-17T00:00:00.000Z')
  primeiro_nome: 'Neena',
  sobrenome: 'Kochhar',
  data_contratacao: ISODate('1989-09-21T00:00:00.000Z')
    primeiro_nome: 'Alexander',
  sobrenome: 'Hunold',
  data_contratacao: ISODate('1990-01-03T00:00:00.000Z')
   primeiro_nome: 'Bruce',
  sobrenome: 'Ernst',
  data_contratacao: ISODate('1991-05-21T00:00:00.000Z')

  db.funcionarios.find(
  {},
  {
    primeiro_nome: 1,
    sobrenome: 1,
    data_contratacao: 1,
    _id: 0
  }
).sort({
  data_contratacao: 1
}).limit(5)

3.6 Quantos funcionários foram contratados na década de 1990 (entre 1990-1999)?
R:37
db.funcionarios.countDocuments({
  data_contratacao: {
    $gte: ISODate("1990-01-01T00:00:00.000Z"),
    $lt: ISODate("2000-01-01T00:00:00.000Z")
  }
})

3.7 Como a média salarial da Momento evoluiu ao longo dos anos? Agrupe por ano de contratação e calcule a média salarial.
R:   media_salarial: Decimal128('14200.00'),
  total_funcionarios: 2,
  ano: 1987
  media_salarial: Decimal128('17000.00'),
  total_funcionarios: 1,
  ano: 1989
   media_salarial: Decimal128('9000.00'),
  total_funcionarios: 1,
  ano: 1990
    media_salarial: Decimal128('6000.00'),
  total_funcionarios: 1,
  ano: 1991
    media_salarial: Decimal128('17000.00'),
  total_funcionarios: 1,
  ano: 1993
    media_salarial: Decimal128('12847.50'),
  total_funcionarios: 8,
  ano: 1994
  media_salarial: Decimal128('5500.00'),
  total_funcionarios: 2,
  ano: 1995
    media_salarial: Decimal128('9750.00'),
  total_funcionarios: 4,
  ano: 1996
   media_salarial: Decimal128('6450.00'),
  total_funcionarios: 10,
  ano: 1997
    media_salarial: Decimal128('5816.666666666666666666666666666667'),
  total_funcionarios: 6,
  ano: 1998
    media_salarial: Decimal128('5150.00'),
  total_funcionarios: 4,
  ano: 1999
    media_salarial: Decimal128('19650.00'),
  total_funcionarios: 1,
  ano: 2000
   media_salarial: Decimal128('13687.50125'),
  total_funcionarios: 8,
  ano: 2024
   media_salarial: 5000,
  total_funcionarios: 1,
  ano: 2026
  

db.funcionarios.aggregate([
  {
    $match: {
      data_contratacao: { $type: "date" }
    }
  },
  {
    $group: {
      _id: { $year: "$data_contratacao" },
      media_salarial: { $avg: "$salario" },
      total_funcionarios: { $sum: 1 }
    }
  },
  {
    $sort: { _id: 1 }
  },
  {
    $project: {
      _id: 0,
      ano: "$_id",
      media_salarial: 1,
      total_funcionarios: 1
    }
  }
])

Nível 4: Operações e Escritórios
4.1 Qual é o custo total de suprimentos em cada escritório? Ordene do mais caro ao mais barato.
R:  escritorio: 'Baxter Building',
  custo_total: Decimal128('20839750.00')
  escritorio: 'Umbrella Corp',
  custo_total: Decimal128('6155000.00')
    escritorio: "Shuri's Palace",
  custo_total: Decimal128('5147785.00')
  escritorio: 'Sala Winter',
  custo_total: Decimal128('4443121.00')
  escritorio: 'Wayne Offices',
  custo_total: Decimal128('4164500.00')
   escritorio: 'House of Mystery escritorios',
  custo_total: Decimal128('140000.00')
    escritorio: 'Arkham Base',
  custo_total: Decimal128('100000.00')

  db.escritorio.aggregate([
  {
    $unwind: "$suprimentos"
  },
  {
    $group: {
      _id: "$_id",
      escritorio: { $first: "$nome" },
      custo_total: {
        $sum: {
          $multiply: [
            "$suprimentos.quantidade",
            "$suprimentos.custo"
          ]
        }
      }
    }
  },
  {
    $sort: {
      custo_total: -1
    }
  },
  {
    $project: {
      _id: 0,
      escritorio: 1,
      custo_total: 1
    }
  }
])

4.2 Qual escritório possui a maior quantidade de diferentes tipos de suprimentos?
R:  tipos_suprimentos: 4,
  escritorio: 'Wayne Offices'

db.escritorio.aggregate([
  {
    $project: {
      nome: 1,
      tipos_suprimentos: {
        $size: {
          $setUnion: [
            {
              $map: {
                input: { $ifNull: ["$suprimentos", []] },
                as: "item",
                in: "$$item.nome"
              }
            },
            []
          ]
        }
      }
    }
  },
  {
    $sort: {
      tipos_suprimentos: -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      escritorio: "$nome",
      tipos_suprimentos: 1
    }
  }
])

4.3 Qual é o suprimento mais caro (considerando preço unitário) em toda a empresa?
R: escritorio: 'Umbrella Corp',
  suprimento: 'Computers',
  preco_unitario: Decimal128('200000.00')
  
db.escritorio.aggregate([
  {
    $unwind: "$suprimentos"
  },
  {
    $sort: {
      "suprimentos.custo": -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      escritorio: "$nome",
      suprimento: "$suprimentos.nome",
      preco_unitario: "$suprimentos.custo"
    }
  }
])

4.4 Calcule o valor total do inventário de suprimentos da empresa (quantidade × preço unitário de todos os itens em todos os escritórios).
R:   valor_total_inventario: Decimal128('40990156.00')

db.escritorio.aggregate([
  {
    $unwind: "$suprimentos"
  },
  {
    $group: {
      _id: null,
      valor_total_inventario: {
        $sum: {
          $multiply: [
            "$suprimentos.quantidade",
            "$suprimentos.custo"
          ]
        }
      }
    }
  },
  {
    $project: {
      _id: 0,
      valor_total_inventario: 1
    }
  }
])

Nível 5: Produtos e Vendas
5.1 Quais produtos foram vendidos pela Momento? Liste todos os produtos únicos.
R:    nome: 'Batarangs Oficiais',
  produto_id: 12
    nome: 'Capacete do Homem-Formiga',
  produto_id: 4
   nome: 'Fake Batarang',
  produto_id: 2
  nome: 'Lança-Teias',
  produto_id: 3
    nome: 'Laço da Honestidade',
  produto_id: 11
  nome: 'Laço da Verdade',
  produto_id: 6
    nome: 'Nulificador Total',
  produto_id: 5
   nome: 'Sabre de Luz (Mace Windu)',
  produto_id: 8
   nome: 'Sentinelas do Bolivar Trask',
  produto_id: 9
nome: 'Uniforme de Moléculas Instáveis',
  produto_id: 10
   nome: 'Uniforme do Superman',
  produto_id: 1
  
db.vendas.aggregate([
  {
    $lookup: {
      from: "produtos",
      localField: "produto_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $group: {
      _id: "$produto._id",
      nome: { $first: "$produto.nome" }
    }
  },
  {
    $project: {
      _id: 0,
      produto_id: "$_id",
      nome: 1
    }
  },
  {
    $sort: {
      nome: 1
    }
  }
])
5.2 Qual é o produto mais vendido (maior quantidade total)?
R:   quantidade_total: 177,
  produto: 'Uniforme do Superman'

db.vendas.aggregate([
  {
    $group: {
      _id: "$produto_id",
      quantidade_total: { $sum: "$quantidade" }
    }
  },
  {
    $lookup: {
      from: "produtos",
      localField: "_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $sort: {
      quantidade_total: -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      produto: "$produto.nome",
      quantidade_total: 1
    }
  }
])
5.3 Qual é o produto menos vendido?
R:  quantidade_total: 3,
  produto: 'Batarangs Oficiais'
  
db.vendas.aggregate([
  {
    $group: {
      _id: "$produto_id",
      quantidade_total: { $sum: "$quantidade" }
    }
  },
  {
    $lookup: {
      from: "produtos",
      localField: "_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $sort: {
      quantidade_total: 1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      produto: "$produto.nome",
      quantidade_total: 1
    }
  }
])

5.4 Pensando na relação quantidade × valor unitário, qual produto gerou mais receita para a empresa?
R:   quantidade_total: 177,
  produto: 'Uniforme do Superman',
  preco_unitario: Decimal128('300.13'),
  receita_total: Decimal128('53123.01')

  db.vendas.aggregate([
  {
    $group: {
      _id: "$produto_id",
      quantidade_total: { $sum: "$quantidade" }
    }
  },
  {
    $lookup: {
      from: "produtos",
      localField: "_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $project: {
      produto: "$produto.nome",
      quantidade_total: 1,
      preco_unitario: "$produto.preco_unitario",
      receita_total: {
        $multiply: [
          "$quantidade_total",
          "$produto.preco_unitario"
        ]
      }
    }
  },
  {
    $sort: {
      receita_total: -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      produto: 1,
      quantidade_total: 1,
      preco_unitario: 1,
      receita_total: 1
    }
  }
])

5.5 Qual é o produto mais caro (maior preço unitário) no catálogo?
R: nome: 'Sabre de Luz (Mace Windu)',
  preco_unitario: Decimal128('990.29')
  db.produtos.find(
  {},
  {
    _id: 0,
    nome: 1,
    preco_unitario: 1
  }
).sort({
  preco_unitario: -1
}).limit(1)

5.6 Qual foi o faturamento total da empresa em vendas?
R: faturamento_total: Decimal128('159186.48')

db.vendas.aggregate([
  {
    $lookup: {
      from: "produtos",
      localField: "produto_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $group: {
      _id: null,
      faturamento_total: {
        $sum: {
          $multiply: [
            "$quantidade",
            "$produto.preco_unitario"
          ]
        }
      }
    }
  },
  {
    $project: {
      _id: 0,
      faturamento_total: 1
    }
  }
])

5.7 Quantas vendas foram realizadas no mês de junho de 2023?
R: 9
db.vendas.countDocuments({
  data_venda: {
    $gte: ISODate("2023-06-01T00:00:00.000Z"),
    $lt: ISODate("2023-07-01T00:00:00.000Z")
  }
})

5.8 Qual vendedor realizou mais vendas (em quantidade de transações)?
R: quantidade_vendas: 4,
  vendedor: 'Jonathon Taylor'
db.vendas.aggregate([
  {
    $group: {
      _id: "$funcionario_id",
      quantidade_vendas: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "funcionarios",
      localField: "_id",
      foreignField: "_id",
      as: "funcionario"
    }
  },
  {
    $unwind: "$funcionario"
  },
  {
    $sort: {
      quantidade_vendas: -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      vendedor: {
        $concat: [
          "$funcionario.primeiro_nome",
          " ",
          "$funcionario.sobrenome"
        ]
      },
      quantidade_vendas: 1
    }
  }
])

5.9 Qual vendedor gerou mais receita para a empresa?
R: receita_total: Decimal128('71582.80'),
  vendedor: 'Jonathon Taylor'

db.vendas.aggregate([
  {
    $lookup: {
      from: "produtos",
      localField: "produto_id",
      foreignField: "_id",
      as: "produto"
    }
  },
  {
    $unwind: "$produto"
  },
  {
    $group: {
      _id: "$funcionario_id",
      receita_total: {
        $sum: {
          $multiply: [
            "$quantidade",
            "$produto.preco_unitario"
          ]
        }
      }
    }
  },
  {
    $lookup: {
      from: "funcionarios",
      localField: "_id",
      foreignField: "_id",
      as: "funcionario"
    }
  },
  {
    $unwind: "$funcionario"
  },
  {
    $sort: {
      receita_total: -1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      vendedor: {
        $concat: [
          "$funcionario.primeiro_nome",
          " ",
          "$funcionario.sobrenome"
        ]
      },
      receita_total: 1
    }
  }
])

Nível 6: Operações de Atualização
A empresa está crescendo e mudanças precisam acontecer!

6.1 Um novo departamento foi criado: Inovações. Ele será alocado no escritório "Wayne Offices". Adicione-o ao banco de dados.

6.2 O departamento de Inovações está sem funcionários. Transfira 2 funcionários do departamento de Tecnologia para Inovações.

6.3 A empresa decidiu dar um aumento de 10% para todos os funcionários do departamento de Tecnologia. Atualize os salários.

6.4 O funcionário "Bruce Ernst" foi promovido a "Senior Web Developer" e recebeu um aumento para $5.000. Atualize suas informações.

6.5 Adicione um novo suprimento ao escritório "Wayne Offices": 15 unidades de "Headsets" com preço unitário de $150 cada.

6.6 Todos os funcionários contratados antes de 1990 estão aposentando. Remova-os do banco de dados (CUIDADO: execute um find antes para ver quantos serão afetados!).

Nível 7: Análise Avançada com Agregações
7.1 Crie um relatório mostrando cada departamento com: nome do departamento, número de funcionários, salário total e média salarial.

7.2 Liste os 3 cargos mais comuns na empresa e quantos funcionários ocupam cada cargo.

7.3 Encontre todos os funcionários que ganham acima da média salarial de seus respectivos departamentos.

7.4 Calcule a taxa de crescimento da empresa por ano (quantos funcionários foram contratados a cada ano).

7.5 Crie um ranking dos vendedores, mostrando: nome do vendedor, número de vendas realizadas e receita total gerada.

7.6 Encontre os produtos que foram vendidos por apenas um vendedor.

Nível 8: Desafios e Otimização
8.1 Desafio de Performance: Encontre todos os funcionários do departamento de Vendas que possuem dependentes (cônjuge OU filhos). Quantas formas diferentes você consegue resolver isso?

8.2 Desafio de Lógica: Encontre funcionários que trabalham em escritórios diferentes do escritório associado ao seu departamento (se houver alguma inconsistência).

8.3 Consulta Complexa: Crie um relatório completo de um escritório mostrando:

Nome do escritório
País
Número de departamentos naquele escritório
Número total de funcionários
Custo total com salários
Custo total com suprimentos
8.4 Desafio de Agregação: Encontre o "departamento mais equilibrado" - aquele com a menor diferença entre o maior e menor salário.

8.5 Text Search: Encontre todos os produtos que contêm a palavra "Uniforme" no nome.

8.6 Date Range: Encontre todas as vendas que ocorreram no segundo trimestre de 2023 (abril-junho).

Nível 9: Desafios Ninja
9.1 Múltiplas Soluções: Encontre todos os funcionários cujo salário está entre $6.000 e $10.000. Resolva de 3 formas diferentes.

9.2 Otimização de Query: Qual destas queries é mais eficiente para encontrar funcionários do departamento de Vendas com salário acima de $7.000? Por quê?

Opção A: db.funcionarios.find({departamento: ObjectId("..."), salario: {$gt: 7000}})
Opção B: db.funcionarios.find({salario: {$gt: 7000}}).filter(doc => doc.departamento == ObjectId("..."))
9.3 Data Quality: Encontre todos os funcionários que NÃO possuem email ou telefone cadastrado.

9.4 Desafio de Lógica Avançada: Crie uma query que encontre "funcionários solitários" - aqueles que são os únicos em seu cargo dentro de seu departamento.

9.5 Pipeline Complexo: Crie um relatório que mostre, para cada país:

Nome do país
Número de escritórios
Número de departamentos
Número de funcionários
Receita total (soma dos salários + custo de suprimentos)
Nível 10: Casos Práticos e Simulações
Cenário 1: Expansão Internacional
A Momento quer abrir um novo escritório no Brasil.

10.1 Crie o novo escritório "Momento Brasil" em São Paulo com suprimentos iniciais de sua escolha.

10.2 Crie um novo departamento "Operações LATAM" vinculado a este escritório.

10.3 Contrate 5 novos funcionários para este departamento (você e 4 colegas de turma).

Cenário 2: Crise Financeira
A empresa precisa reduzir custos!

10.4 Identifique os 3 escritórios com maiores custos de suprimentos.

10.5 Sugira onde cortar custos: liste suprimentos com quantidade acima de 50 unidades.

10.6 Calcule quanto a empresa economizaria se reduzisse 20% dos salários acima de $15.000 (apenas calcule, não execute).

Cenário 3: Auditoria de Vendas
O departamento financeiro suspeita de irregularidades.

10.7 Encontre vendas que não possuem vendedor associado.

10.8 Verifique se há vendedores registrados nas vendas que não existem na coleção de funcionários.

10.9 Identifique produtos que aparecem nas vendas mas com preços unitários muito diferentes entre transações (variação > 10%).

Cenário 4: Relatório Executivo
O CEO pediu um dashboard completo.

10.10 Crie UMA ÚNICA query de agregação que retorne:

Total de funcionários
Custo total com salários
Número de departamentos
Número de escritórios
Receita total de vendas
Produto mais vendido
