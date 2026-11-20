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

# Nível 4: Operações e Escritórios

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

# Nível 5: Produtos e Vendas

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

# Nível 6: Operações de Atualização
A empresa está crescendo e mudanças precisam acontecer!

6.1 Um novo departamento foi criado: Inovações. Ele será alocado no escritório "Wayne Offices". Adicione-o ao banco de dados.
R: db.departamentos.insertOne({
  nome: "Inovações",
  escritorio_id: 1800
})

6.2 O departamento de Inovações está sem funcionários. Transfira 2 funcionários do departamento de Tecnologia para Inovações.
R: db.funcionarios.updateMany(
  {
    _id: {
      $in: [216, 214]
    }
  },
  {
    $set: {
      departamento_id: ('6affcea97b16be3ebdd858fc')
    }
  }
)

6.3 A empresa decidiu dar um aumento de 10% para todos os funcionários do departamento de Tecnologia. Atualize os salários.
R: db.funcionarios.updateMany(
  { departamento_id: 6 },
  {
    $mul: {
      salario: NumberDecimal("1.10")
    }
  }
)

6.4 O funcionário "Bruce Ernst" foi promovido a "Senior Web Developer" e recebeu um aumento para $5.000. Atualize suas informações.
R:  primeiro_nome: 'Bruce',
  sobrenome: 'Ernst',
  salario: Decimal128('7000.00'),
  cargo: 'Senior Web Developer'
  
db.cargos.insertOne({
  nome: "Senior Web Developer",
  salario_minimo: NumberDecimal("7000.00"),
  salario_maximo: NumberDecimal("15000.00")
})

6.5 Adicione um novo suprimento ao escritório "Wayne Offices": 15 unidades de "Headsets" com preço unitário de $150 cada.
R: db.escritorio.updateOne(
  { nome: "Wayne Offices" },
  {
    $push: {
      suprimentos: {
        nome: "Headsets",
        quantidade: 15,
        custo: NumberDecimal("150.00")
      }
    }
  }
)

6.6 Todos os funcionários contratados antes de 1990 estão aposentando. Remova-os do banco de dados (CUIDADO: execute um find antes para ver quantos serão afetados!).
R: primeiro_nome: 'Madeleine',
  sobrenome: 'Prior',
  data_contratacao: ISODate('1987-06-17T00:00:00.000Z')

primeiro_nome: 'Jennifer',
  sobrenome: 'Whalen',
  data_contratacao: ISODate('1987-09-17T00:00:00.000Z')
db.funcionarios.find(

 primeiro_nome: 'Neena',
  sobrenome: 'Kochhar',
  data_contratacao: ISODate('1989-09-21T00:00:00.000Z')

  3 funcionários serão excluidos
  {
    data_contratacao: {
      $lt: ISODate("1990-01-01T00:00:00.000Z")
    }
  },
  {
    primeiro_nome: 1,
    sobrenome: 1,
    data_contratacao: 1,
    _id: 0
  }
).sort({
  data_contratacao: 1
})

db.funcionarios.countDocuments({
  data_contratacao: {
    $lt: ISODate("1990-01-01T00:00:00.000Z")
  }
})

db.funcionarios.deleteMany({
  data_contratacao: {
    $lt: ISODate("1990-01-01T00:00:00.000Z")
  }
})

# Nível 7: Análise Avançada com Agregações

7.1 Crie um relatório mostrando cada departamento com: nome do departamento, número de funcionários, salário total e média salarial.
R: 
  numero_funcionarios: 4,
  salario_total: Decimal128('86630.00'),
  media_salarial: Decimal128('21657.50'),
  departamento: 'Tecnologias Avançadas'
   numero_funcionarios: 7,
  salario_total: Decimal128('59600.01'),
  media_salarial: Decimal128('8514.287142857142857142857142857143'),
  departamento: 'Finanças'
  numero_funcionarios: 5,
  salario_total: Decimal128('51500.00'),
  media_salarial: Decimal128('10300.00'),
  departamento: 'Vendas'
    numero_funcionarios: 2,
  salario_total: Decimal128('41300.00'),
  media_salarial: Decimal128('20650.00'),
  departamento: 'Biotecnologia'
   numero_funcionarios: 7,
  salario_total: Decimal128('41200.00'),
  media_salarial: Decimal128('5885.714285714285714285714285714286'),
  departamento: 'Transportes'
  numero_funcionarios: 6,
  salario_total: Decimal128('39930.0000'),
  media_salarial: Decimal128('6655.0000'),
  departamento: 'Tecnologia'
    numero_funcionarios: 6,
  salario_total: Decimal128('24900.00'),
  media_salarial: Decimal128('4150.00'),
  departamento: 'Suprimentos'
   numero_funcionarios: 2,
  salario_total: Decimal128('20000.00'),
  media_salarial: Decimal128('10000.00'),
  departamento: 'Relações Públicas'
  numero_funcionarios: 2,
  salario_total: Decimal128('19000.00'),
  media_salarial: Decimal128('9500.00'),
  departamento: 'Marketing'
    numero_funcionarios: 1,
  salario_total: Decimal128('17000.00'),
  media_salarial: Decimal128('17000.00'),
  departamento: 'Executivo'
   numero_funcionarios: 1,
  salario_total: Decimal128('12000.00'),
  media_salarial: Decimal128('12000.00'),
  departamento: 'Contabilidade'
    numero_funcionarios: 1,
  salario_total: Decimal128('6500.00'),
  media_salarial: Decimal128('6500.00'),
  departamento: 'Recursos Humanos'

db.funcionarios.aggregate([
  {
    $group: {
      _id: "$departamento_id",
      numero_funcionarios: { $sum: 1 },
      salario_total: { $sum: "$salario" },
      media_salarial: { $avg: "$salario" }
    }
  },
  {
    $lookup: {
      from: "departamentos",
      localField: "_id",
      foreignField: "_id",
      as: "departamento"
    }
  },
  {
    $unwind: "$departamento"
  },
  {
    $project: {
      _id: 0,
      departamento: "$departamento.nome",
      numero_funcionarios: 1,
      salario_total: 1,
      media_salarial: 1
    }
  },
  {
    $sort: {
      salario_total: -1
    }
  }
])

7.2 Liste os 3 cargos mais comuns na empresa e quantos funcionários ocupam cada cargo.
R: quantidade_funcionarios: 6,
  cargo: 'Web Developer'
   quantidade_funcionarios: 6,
  cargo: 'Cientista'
  quantidade_funcionarios: 5,
  cargo: 'Purchasing Clerk'

  db.funcionarios.aggregate([
  {
    $group: {
      _id: "$cargo_id",
      quantidade_funcionarios: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "cargos",
      localField: "_id",
      foreignField: "_id",
      as: "cargo"
    }
  },
  {
    $unwind: "$cargo"
  },
  {
    $project: {
      _id: 0,
      cargo: "$cargo.nome",
      quantidade_funcionarios: 1
    }
  },
  {
    $sort: {
      quantidade_funcionarios: -1
    }
  },
  {
    $limit: 3
  }
])

7.3 Encontre todos os funcionários que ganham acima da média salarial de seus respectivos departamentos.
R:  primeiro_nome: 'Spock',
  sobrenome: 'Sarek',
  salario: Decimal128('23000.00'),
  departamento: 'Biotecnologia',
  media_salarial_departamento: Decimal128('20650.00')
   primeiro_nome: 'Lucius',
  sobrenome: 'Fox',
  salario: Decimal128('12000.00'),
  departamento: 'Finanças',
  media_salarial_departamento: Decimal128('8514.287142857142857142857142857143')
   primeiro_nome: 'Daniel',
  sobrenome: 'Faviet',
  salario: Decimal128('9000.00'),
  departamento: 'Finanças',
  media_salarial_departamento: Decimal128('8514.287142857142857142857142857143')
   primeiro_nome: 'Elisabeth',
  sobrenome: 'Braddock',
  salario: Decimal128('13000.00'),
  departamento: 'Marketing',
  media_salarial_departamento: Decimal128('9500.00')
   primeiro_nome: 'Den',
  sobrenome: 'Raphaely',
  salario: Decimal128('11000.00'),
  departamento: 'Suprimentos',
  media_salarial_departamento: Decimal128('4150.00')
    primeiro_nome: 'Alexander',
  sobrenome: 'Hunold',
  salario: Decimal128('9900.0000'),
  departamento: 'Tecnologia',
  media_salarial_departamento: Decimal128('6655.0000')
    primeiro_nome: 'Bruce',
  sobrenome: 'Ernst',
  salario: Decimal128('7700.0000'),
  departamento: 'Tecnologia',
  media_salarial_departamento: Decimal128('6655.0000')
   primeiro_nome: 'Peter',
  sobrenome: 'Parker',
  salario: Decimal128('7150.0000'),
  departamento: 'Tecnologia',
  media_salarial_departamento: Decimal128('6655.0000')
  primeiro_nome: 'Shuri',
  sobrenome: 'Udaku',
  salario: Decimal128('25000.00'),
  departamento: 'Tecnologias Avançadas',
  media_salarial_departamento: Decimal128('21657.50')
    primeiro_nome: 'Nancy',
  sobrenome: 'Constantine',
  salario: Decimal128('23980.00'),
  departamento: 'Tecnologias Avançadas',
  media_salarial_departamento: Decimal128('21657.50')
    primeiro_nome: 'Adam',
  sobrenome: 'Fripp',
  salario: Decimal128('8200.00'),
  departamento: 'Transportes',
  media_salarial_departamento: Decimal128('5885.714285714285714285714285714286')
    primeiro_nome: 'Matthew',
  sobrenome: 'Weiss',
  salario: Decimal128('8000.00'),
  departamento: 'Transportes',
  media_salarial_departamento: Decimal128('5885.714285714285714285714285714286')
    primeiro_nome: 'Payam',
  sobrenome: 'Kaufling',
  salario: Decimal128('7900.00'),
  departamento: 'Transportes',
  media_salarial_departamento: Decimal128('5885.714285714285714285714285714286')
    primeiro_nome: 'Shanta',
  sobrenome: 'Vollman',
  salario: Decimal128('6500.00'),
  departamento: 'Transportes',
  media_salarial_departamento: Decimal128('5885.714285714285714285714285714286')
    primeiro_nome: 'John',
  sobrenome: 'Russell',
  salario: Decimal128('14000.00'),
  departamento: 'Vendas',
  media_salarial_departamento: Decimal128('10300.00')
   primeiro_nome: 'Karen',
  sobrenome: 'Partners',
  salario: Decimal128('13500.00'),
  departamento: 'Vendas',
  media_salarial_departamento: Decimal128('10300.00')
  db.funcionarios.aggregate([
  {
    $lookup: {
      from: "funcionarios",
      let: {
        departamento_atual: "$departamento_id"
      },
      pipeline: [
        {
          $match: {
            $expr: {
              $eq: [
                "$departamento_id",
                "$$departamento_atual"
              ]
            }
          }
        },
        {
          $group: {
            _id: null,
            media_departamento: { $avg: "$salario" }
          }
        }
      ],
      as: "media_depto"
    }
  },
  {
    $unwind: "$media_depto"
  },
  {
    $match: {
      $expr: {
        $gt: [
          "$salario",
          "$media_depto.media_departamento"
        ]
      }
    }
  },
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "departamento"
    }
  },
  {
    $unwind: "$departamento"
  },
  {
    $project: {
      _id: 0,
      primeiro_nome: 1,
      sobrenome: 1,
      salario: 1,
      departamento: "$departamento.nome",
      media_salarial_departamento: "$media_depto.media_departamento"
    }
  },
  {
    $sort: {
      departamento: 1,
      salario: -1
    }
  }
])

7.4 Calcule a taxa de crescimento da empresa por ano (quantos funcionários foram contratados a cada ano).
R:  contratacoes: 1,
  ano: 1990
  contratacoes: 1,
  ano: 1991
    contratacoes: 1,
  ano: 1993
   contratacoes: 8,
  ano: 1994
  contratacoes: 2,
  ano: 1995
    contratacoes: 4,
  ano: 1996
   contratacoes: 10,
  ano: 1997
  contratacoes: 6,
  ano: 1998
  contratacoes: 4,
  ano: 1999
  contratacoes: 1,
  ano: 2000
    contratacoes: 8,
  ano: 2024
   contratacoes: 1,
  ano: 2026
db.funcionarios.aggregate([
  {
    $match: {
      data_contratacao: { $type: "date" }
    }
  },
  {
    $group: {
      _id: {
        $year: "$data_contratacao"
      },
      contratacoes: { $sum: 1 }
    }
  },
  {
    $sort: {
      _id: 1
    }
  },
  {
    $project: {
      _id: 0,
      ano: "$_id",
      contratacoes: 1
    }
  }
])

7.5 Crie um ranking dos vendedores, mostrando: nome do vendedor, número de vendas realizadas e receita total gerada.
R:   numero_vendas: 4,
  receita_total: Decimal128('71582.80'),
  vendedor: 'Jonathon Taylor'
   numero_vendas: 4,
  receita_total: Decimal128('30563.65'),
  vendedor: 'Karen Partners'
   numero_vendas: 3,
  receita_total: Decimal128('27238.26'),
  vendedor: 'Kimberly Grant'
   numero_vendas: 4,
  receita_total: Decimal128('22552.92'),
  vendedor: 'John Russell'
   numero_vendas: 2,
  receita_total: Decimal128('7248.85'),
  vendedor: 'Jack Livingston'

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
      numero_vendas: { $sum: 1 },
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
    $project: {
      _id: 0,
      vendedor: {
        $concat: [
          "$funcionario.primeiro_nome",
          " ",
          "$funcionario.sobrenome"
        ]
      },
      numero_vendas: 1,
      receita_total: 1
    }
  },
  {
    $sort: {
      receita_total: -1
    }
  }
])

7.6 Encontre os produtos que foram vendidos por apenas um vendedor.
R:   produto: 'Batarangs Oficiais',
  vendedor: 'Jonathon Taylor'
   produto: 'Fake Batarang',
  vendedor: 'Kimberly Grant'
   produto: 'Laço da Honestidade',
  vendedor: 'Karen Partners'
  produto: 'Sabre de Luz (Mace Windu)',
  vendedor: 'Kimberly Grant'
   produto: 'Sentinelas do Bolivar Trask',
  vendedor: 'Kimberly Grant'
    produto: 'Uniforme de Moléculas Instáveis',
  vendedor: 'John Russell'

  db.vendas.aggregate([
  {
    $group: {
      _id: "$produto_id",
      vendedores: {
        $addToSet: "$funcionario_id"
      }
    }
  },
  {
    $match: {
      $expr: {
        $eq: [
          { $size: "$vendedores" },
          1
        ]
      }
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
      _id: 0,
      produto: "$produto.nome",
      vendedor_id: {
        $arrayElemAt: [
          "$vendedores",
          0
        ]
      }
    }
  },
  {
    $lookup: {
      from: "funcionarios",
      localField: "vendedor_id",
      foreignField: "_id",
      as: "funcionario"
    }
  },
  {
    $unwind: "$funcionario"
  },
  {
    $project: {
      produto: 1,
      vendedor: {
        $concat: [
          "$funcionario.primeiro_nome",
          " ",
          "$funcionario.sobrenome"
        ]
      }
    }
  },
  {
    $sort: {
      produto: 1
    }
  }
])

# Nível 8: Desafios e Otimização
8.1 Desafio de Performance: Encontre todos os funcionários do departamento de Vendas que possuem dependentes (cônjuge OU filhos). Quantas formas diferentes você consegue resolver isso?
R:

db.funcionarios.find({
  departamento_id: { $in: [15, 16, 17] },
  dependentes: {
    $elemMatch: {
      relacionamento: {
        $in: ["Cônjuge", "Filha(o)"]
      }
    }
  }
})

db.funcionarios.find({
  departamento_id: { $in: [15, 16, 17] },
  $or: [
    {
      dependentes: {
        $elemMatch: { relacionamento: "Cônjuge" }
      }
    },
    {
      dependentes: {
        $elemMatch: { relacionamento: "Filha(o)" }
      }
    }
  ]
})

db.funcionarios.find({
  departamento_id: { $in: [15, 16, 17] },
  "dependentes.relacionamento": {
    $in: ["Cônjuge", "Filha(o)"]
  }
})
8.2 Desafio de Lógica: Encontre funcionários que trabalham em escritórios diferentes do escritório associado ao seu departamento (se houver alguma inconsistência).
R: não há funcionarios que trabalham em escritórios diferentes
db.funcionarios.aggregate([
  {
    $match: {
      departamento_id: 6,
      escritorio_id: { $exists: true }
    }
  },
  {
    $lookup: {
      from: "departamentos",
      localField: "departamento_id",
      foreignField: "_id",
      as: "departamento"
    }
  },
  {
    $unwind: "$departamento"
  },
  {
    $match: {
      $expr: {
        $ne: [
          "$escritorio_id",
          "$departamento.escritorio_id"
        ]
      }
    }
  },
  {
    $project: {
      _id: 1,
      primeiro_nome: 1,
      sobrenome: 1,
      departamento: "$departamento.nome",
      escritorio_funcionario: "$escritorio_id",
      escritorio_departamento: "$departamento.escritorio_id"
    }
  }
])


8.3 Consulta Complexa: Crie um relatório completo de um escritório mostrando:

Nome do escritório
País
Número de departamentos naquele escritório
Número total de funcionários
Custo total com salários
Custo total com suprimentos
R:  _id: 1700,
  nome: 'Sala Winter',
  numero_departamentos: 5,
  total_funcionarios: 15,
  custo_salarios: Decimal128('113500.01'),
  custo_suprimentos: Decimal128('4443121.00')
  _id: 1900,
  nome: 'House of Mystery escritorios',
  numero_departamentos: 1,
  total_funcionarios: 4,
  custo_salarios: Decimal128('86630.00'),
  custo_suprimentos: Decimal128('140000.00')
   _id: 2500,
  nome: 'Umbrella Corp',
  numero_departamentos: 1,
  total_funcionarios: 5,
  custo_salarios: Decimal128('51500.00'),
  custo_suprimentos: Decimal128('6155000.00')
    _id: 3900,
  nome: 'Baxter Building',
  numero_departamentos: 1,
  total_funcionarios: 2,
  custo_salarios: Decimal128('41300.00'),
  custo_suprimentos: Decimal128('20839750.00')
    _id: 1500,
  nome: 'Stark Tower',
  numero_departamentos: 1,
  total_funcionarios: 7,
  custo_salarios: Decimal128('41200.00'),
  custo_suprimentos: 0
   _id: 1400,
  nome: 'Wayne Offices',
  numero_departamentos: 1,
  total_funcionarios: 6,
  custo_salarios: Decimal128('39930.0000'),
  custo_suprimentos: Decimal128('4166750.00')
    _id: 2700,
  nome: 'Archies Mad House',
  numero_departamentos: 1,
  total_funcionarios: 2,
  custo_salarios: Decimal128('20000.00'),
  custo_suprimentos: 0
   _id: 1800,
  nome: 'Arkham Base',
  numero_departamentos: 2,
  total_funcionarios: 2,
  custo_salarios: Decimal128('19000.00'),
  custo_suprimentos: Decimal128('100000.00')
    _id: 2400,
  nome: "Shuri's Palace",
  numero_departamentos: 1,
  total_funcionarios: 1,
  custo_salarios: Decimal128('6500.00'),
  custo_suprimentos: Decimal128('5147785.00')


db.escritorio.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "_id",
      foreignField: "escritorio_id",
      as: "departamentos"
    }
  },
  {
    $lookup: {
      from: "funcionarios",
      let: {
        id_escritorio: "$_id"
      },
      pipeline: [
        {
          $lookup: {
            from: "departamentos",
            localField: "departamento_id",
            foreignField: "_id",
            as: "departamento"
          }
        },
        {
          $unwind: "$departamento"
        },
        {
          $match: {
            $expr: {
              $eq: [
                "$departamento.escritorio_id",
                "$$id_escritorio"
              ]
            }
          }
        },
        {
          $group: {
            _id: null,
            total_funcionarios: { $sum: 1 },
            custo_salarios: { $sum: "$salario" }
          }
        }
      ],
      as: "dados_funcionarios"
    }
  },
  {
    $project: {
      nome: 1,
      pais: 1,
      numero_departamentos: {
        $size: "$departamentos"
      },
      total_funcionarios: {
        $ifNull: [
          { $arrayElemAt: ["$dados_funcionarios.total_funcionarios", 0] },
          0
        ]
      },
      custo_salarios: {
        $ifNull: [
          { $arrayElemAt: ["$dados_funcionarios.custo_salarios", 0] },
          0
        ]
      },
      custo_suprimentos: {
        $sum: {
          $map: {
            input: { $ifNull: ["$suprimentos", []] },
            as: "item",
            in: {
              $multiply: [
                "$$item.quantidade",
                "$$item.custo"
              ]
            }
          }
        }
      }
    }
  },
  {
    $sort: {
      custo_salarios: -1
    }
  }
])
8.4 Desafio de Agregação: Encontre o "departamento mais equilibrado" - aquele com a menor diferença entre o maior e menor salário.
R:   maior_salario: Decimal128('17000.00'),
  menor_salario: Decimal128('17000.00'),
  diferenca_salarial: Decimal128('0.00'),
  departamento: 'Executivo'
  
db.funcionarios.aggregate([
  {
    $group: {
      _id: "$departamento_id",
      maior_salario: { $max: "$salario" },
      menor_salario: { $min: "$salario" }
    }
  },
  {
    $project: {
      diferenca_salarial: {
        $subtract: [
          "$maior_salario",
          "$menor_salario"
        ]
      },
      maior_salario: 1,
      menor_salario: 1
    }
  },
  {
    $lookup: {
      from: "departamentos",
      localField: "_id",
      foreignField: "_id",
      as: "departamento"
    }
  },
  {
    $unwind: "$departamento"
  },
  {
    $sort: {
      diferenca_salarial: 1
    }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      departamento: "$departamento.nome",
      maior_salario: 1,
      menor_salario: 1,
      diferenca_salarial: 1
    }
  }
])

8.5 Text Search: Encontre todos os produtos que contêm a palavra "Uniforme" no nome.
R:   _id: 1,
  nome: 'Uniforme do Superman',
  preco_unitario: Decimal128('300.13')
   _id: 10,
  nome: 'Uniforme de Moléculas Instáveis',
  preco_unitario: Decimal128('158.29')
  
db.produtos.find({
  nome: {
    $regex: "Uniforme",
    $options: "i"
  }
})

8.6 Date Range: Encontre todas as vendas que ocorreram no segundo trimestre de 2023 (abril-junho).
R:   _id: 35864200,
  produto_id: 3,
  quantidade: 19,
  data_venda: ISODate('2023-06-05T00:00:00.000Z'),
  funcionario_id: 176
    _id: 460333211,
  produto_id: 6,
  quantidade: 15,
  data_venda: ISODate('2023-06-10T00:00:00.000Z'),
  funcionario_id: 177
  _id: 23192902,
  produto_id: 5,
  quantidade: 17,
  data_venda: ISODate('2023-06-13T00:00:00.000Z'),
  funcionario_id: 146
   _id: 23112902,
  produto_id: 6,
  quantidade: 13,
  data_venda: ISODate('2023-06-14T00:00:00.000Z'),
  funcionario_id: 146
   _id: 58923150,
  produto_id: 8,
  quantidade: 22,
  data_venda: ISODate('2023-06-15T00:00:00.000Z'),
  funcionario_id: 178
    _id: 11508951,
  produto_id: 1,
  quantidade: 10,
  data_venda: ISODate('2023-06-19T00:00:00.000Z'),
  funcionario_id: 145
   _id: 19290129,
  produto_id: 9,
  quantidade: 14,
  data_venda: ISODate('2023-06-20T00:00:00.000Z'),
  funcionario_id: 178
  _id: 29290213,
  produto_id: 4,
  quantidade: 55,
  data_venda: ISODate('2023-06-21T00:00:00.000Z'),
  funcionario_id: 145
   _id: 12508951,
  produto_id: 10,
  quantidade: 13,
  data_venda: ISODate('2023-06-29T00:00:00.000Z'),
  funcionario_id: 145
  
  
db.vendas.find({
  data_venda: {
    $gte: ISODate("2023-04-01T00:00:00.000Z"),
    $lt: ISODate("2023-07-01T00:00:00.000Z")
  }
})

# Nível 9: Desafios Ninja
9.1 Múltiplas Soluções: Encontre todos os funcionários cujo salário está entre $6.000 e $10.000. Resolva de 3 formas diferentes.
R:

db.funcionarios.find({
  salario: {
    $gte: 6000,
    $lte: 10000
  }
})
db.funcionarios.find({
  $and: [
    { salario: { $gte: 6000 } },
    { salario: { $lte: 10000 } }
  ]
})

db.funcionarios.find({
  $expr: {
    $and: [
      { $gte: ["$salario", 6000] },
      { $lte: ["$salario", 10000] }
    ]
  }
})



 9.2 Otimização de Query: Qual destas queries é mais eficiente para encontrar funcionários do departamento de Vendas com salário acima de $7.000? Por quê?

Opção A: db.funcionarios.find({departamento: ObjectId("..."), salario: {$gt: 7000}})
Opção B: db.funcionarios.find({salario: {$gt: 7000}}).filter(doc => doc.departamento == ObjectId("..."))


R: Opção A
db.funcionarios.find({
  departamento_id: 8,
  salario: { $gt: 7000 }
})

9.3 Data Quality: Encontre todos os funcionários que NÃO possuem email ou telefone cadastrado.
R:

db.funcionarios.find({
  $or: [
    { email: { $exists: false } },
    { email: null },
    { email: "" },
    { telefone: { $exists: false } },
    { telefone: null },
    { telefone: "" }
  ]
})

9.4 Desafio de Lógica Avançada: Crie uma query que encontre "funcionários solitários" - aqueles que são os únicos em seu cargo dentro de seu departamento.
R:  quantidade: 1,
  primeiro_nome: 'Bruce',
  sobrenome: 'Ernst',
  departamento: 'Tecnologia',
  cargo: 'Senior Web Developer'
    quantidade: 1,
  primeiro_nome: 'Lex',
  sobrenome: "T'Challa",
  departamento: 'Executivo',
  cargo: 'Diretor de Administração, Contabilidade e Recursos'
    quantidade: 1,
  primeiro_nome: 'Den',
  sobrenome: 'Raphaely',
  departamento: 'Suprimentos',
  cargo: 'Gerente de Recursos'
  quantidade: 1,
  primeiro_nome: 'Susan',
  sobrenome: 'Mavris',
  departamento: 'Recursos Humanos',
  cargo: 'Assistente de Recursos'
  quantidade: 1,
  primeiro_nome: 'Elisabeth',
  sobrenome: 'Braddock',
  departamento: 'Marketing',
  cargo: 'CMO'
   quantidade: 1,
  primeiro_nome: 'Irene',
  sobrenome: 'Mikkilineni',
  departamento: 'Transportes',
  cargo: 'Estoquista'
    quantidade: 1,
  primeiro_nome: 'Sabrina Portela',
  sobrenome: 'Spellman',
  departamento: 'Finanças',
  cargo: 'Cientista'
   quantidade: 1,
  primeiro_nome: 'Pat',
  sobrenome: 'Ferreira',
  departamento: 'Marketing',
  cargo: 'Representante de Vendas para a América Latina'
  quantidade: 1,
  primeiro_nome: 'Shelley',
  sobrenome: 'Higgins',
  departamento: 'Contabilidade',
  cargo: 'Gerente de Contabilidade'
   quantidade: 1,
  primeiro_nome: 'Lucius',
  sobrenome: 'Fox',
  departamento: 'Finanças',
  cargo: 'CFO'
  quantidade: 1,
  primeiro_nome: 'Nancy',
  sobrenome: 'Constantine',
  departamento: 'Tecnologias Avançadas',
  cargo: 'Web Developer'
  

db.funcionarios.aggregate([
  {
    $group: {
      _id: {
        departamento_id: "$departamento_id",
        cargo_id: "$cargo_id"
      },
      quantidade: { $sum: 1 },
      funcionarios: {
        $push: {
          _id: "$_id",
          primeiro_nome: "$primeiro_nome",
          sobrenome: "$sobrenome"
        }
      }
    }
  },
  {
    $match: {
      quantidade: 1
    }
  },
  {
    $unwind: "$funcionarios"
  },
  {
    $lookup: {
      from: "departamentos",
      localField: "_id.departamento_id",
      foreignField: "_id",
      as: "departamento"
    }
  },
  {
    $lookup: {
      from: "cargos",
      localField: "_id.cargo_id",
      foreignField: "_id",
      as: "cargo"
    }
  },
  {
    $project: {
      _id: 0,
      primeiro_nome: "$funcionarios.primeiro_nome",
      sobrenome: "$funcionarios.sobrenome",
      departamento: {
        $arrayElemAt: ["$departamento.nome", 0]
      },
      cargo: {
        $arrayElemAt: ["$cargo.nome", 0]
      },
      quantidade: 1
    }
  }
])
9.5 Pipeline Complexo: Crie um relatório que mostre, para cada país:

Nome do país
Número de escritórios
Número de departamentos
Número de funcionários
Receita total (soma dos salários + custo de suprimentos)
# Nível 10: Casos Práticos e Simulações
# Cenário 1: Expansão Internacional
A Momento quer abrir um novo escritório no Brasil.

R:  numero_escritorios: 9,
  numero_departamentos: 14,
  numero_funcionarios: 44,
  total_salarios: Decimal128('419560.0100'),
  custo_suprimentos: Decimal128('40992406.00'),
  pais: null,
  receita_total: Decimal128('41411966.0100')

db.escritorio.aggregate([
  {
    $lookup: {
      from: "departamentos",
      localField: "_id",
      foreignField: "escritorio_id",
      as: "departamentos"
    }
  },
  {
    $lookup: {
      from: "funcionarios",
      let: {
        id_escritorio: "$_id"
      },
      pipeline: [
        {
          $lookup: {
            from: "departamentos",
            localField: "departamento_id",
            foreignField: "_id",
            as: "departamento"
          }
        },
        {
          $unwind: "$departamento"
        },
        {
          $match: {
            $expr: {
              $eq: [
                "$departamento.escritorio_id",
                "$$id_escritorio"
              ]
            }
          },
        },
        {
          $group: {
            _id: null,
            total_funcionarios: { $sum: 1 },
            total_salarios: { $sum: "$salario" }
          }
        }
      ],
      as: "funcionarios"
    }
  },
  {
    $project: {
      pais: 1,
      numero_escritorios: { $literal: 1 },
      departamentos: { $size: "$departamentos" },
      funcionarios: {
        $ifNull: [
          { $arrayElemAt: ["$funcionarios.total_funcionarios", 0] },
          0
        ]
      },
      total_salarios: {
        $ifNull: [
          { $arrayElemAt: ["$funcionarios.total_salarios", 0] },
          0
        ]
      },
      custo_suprimentos: {
        $sum: {
          $map: {
            input: { $ifNull: ["$suprimentos", []] },
            as: "item",
            in: {
              $multiply: [
                "$$item.quantidade",
                "$$item.custo"
              ]
            }
          }
        }
      }
    }
  },
  {
    $group: {
      _id: "$pais",
      numero_escritorios: { $sum: "$numero_escritorios" },
      numero_departamentos: { $sum: "$departamentos" },
      numero_funcionarios: { $sum: "$funcionarios" },
      total_salarios: { $sum: "$total_salarios" },
      custo_suprimentos: { $sum: "$custo_suprimentos" }
    }
  },
  {
    $project: {
      _id: 0,
      pais: "$_id",
      numero_escritorios: 1,
      numero_departamentos: 1,
      numero_funcionarios: 1,
      total_salarios: 1,
      custo_suprimentos: 1,
      receita_total: {
        $add: [
          "$total_salarios",
          "$custo_suprimentos"
        ]
      }
    }
  },
  {
    $sort: {
      receita_total: -1
    }
  }
])

10.1 Crie o novo escritório "Momento Brasil" em São Paulo com suprimentos iniciais de sua escolha.
R:
db.escritorio.insertOne({
  _id: 999,
  nome: "Momento Brasil",
  pais: "Brasil",
  cidade: "São Paulo",
  endereco: "Av. Paulista, 1000",
  suprimentos: [
    {
      _id: 1,
      nome: "Papel A4",
      quantidade: 100,
      custo: NumberDecimal("32.50")
    },
    {
      _id: 2,
      nome: "Caneta Azul",
      quantidade: 80,
      custo: NumberDecimal("2.50")
    },
    {
      _id: 3,
      nome: "Cadeira de Escritório",
      quantidade: 10,
      custo: NumberDecimal("450.00")
    },
    {
      _id: 4,
      nome: "Monitor",
      quantidade: 5,
      custo: NumberDecimal("1200.00")
    },
    {
      _id: 5,
      nome: "Teclado",
      quantidade: 10,
      custo: NumberDecimal("150.00")
    }
  ]
})

10.2 Crie um novo departamento "Operações LATAM" vinculado a este escritório.
R:
db.departamentos.insertOne({
  _id: 999,
  nome: "Operações LATAM",
  escritorio_id: 999
})

10.3 Contrate 5 novos funcionários para este departamento (você e 4 colegas de turma).
R:
db.funcionarios.insertMany([
  {
    _id: 301,
    primeiro_nome: "Wesley",
    sobrenome: "Safadão",
    email: "WesleySafadão01@momento.org",
    telefone: "11900000001",
    data_contratacao: ISODate("2026-11-20"),
    cargo_id: 1,
    salario: NumberDecimal("7000.00"),
    gerente_id: null,
    departamento_id: 999,
    dependentes: []
  },
  {
    _id: 902,
    primeiro_nome: "Maryl",
    sobrenome: "streep",
    email: "Miranda02@momento.org",
    telefone: "11900000002",
    data_contratacao: ISODate("2026-11-20"),
    cargo_id: 1,
    salario: NumberDecimal("7000.00"),
    gerente_id: 9001,
    departamento_id: 999,
    dependentes: []
  },
  {
    _id: 9003,
    primeiro_nome: "Angelina",
    sobrenome: "Jolie",
    email: "TombRaider03@momento.org",
    telefone: "11900000003",
    data_contratacao: ISODate("2026-11-20"),
    cargo_id: 1,
    salario: NumberDecimal("7000.00"),
    gerente_id: 9001,
    departamento_id: 999,
    dependentes: []
  },
  {
    _id: 9004,
    primeiro_nome: "Logan",
    sobrenome: "Wolverine",
    email: "Wolverine04@momento.org",
    telefone: "11900000004",
    data_contratacao: ISODate("2026-11-20"),
    cargo_id: 1,
    salario: NumberDecimal("7000.00"),
    gerente_id: 9001,
    departamento_id: 999,
    dependentes: []
  },
  {
    _id: 9005,
    primeiro_nome: "Shirley",
    sobrenome: "Maddison",
    email: "ShirMad05@momento.org",
    telefone: "11900000005",
    data_contratacao: ISODate("2026-11-20"),
    cargo_id: 1,
    salario: NumberDecimal("7000.00"),
    gerente_id: 9001,
    departamento_id: 999,
    dependentes: []
  }
])

# Cenário 2: Crise Financeira
A empresa precisa reduzir custos!

10.4 Identifique os 3 escritórios com maiores custos de suprimentos.
R:_id: 3900,
  nome: 'Baxter Building',
  custo_suprimentos: Decimal128('20839750.00')
   _id: 2500,
  nome: 'Umbrella Corp',
  custo_suprimentos: Decimal128('6155000.00')
   _id: 2400,
  nome: "Shuri's Palace",
  custo_suprimentos: Decimal128('5147785.00')
  

db.escritorio.aggregate([
  {
    $project: {
      nome: 1,
      custo_suprimentos: {
        $sum: {
          $map: {
            input: { $ifNull: ["$suprimentos", []] },
            as: "item",
            in: {
              $multiply: [
                "$$item.quantidade",
                "$$item.custo"
              ]
            }
          }
        }
      }
    }
  },
  {
    $sort: {
      custo_suprimentos: -1
    }
  },
  {
    $limit: 3
  }
])

10.5 Sugira onde cortar custos: liste suprimentos com quantidade acima de 50 unidades.
R:  escritorio: 'Sala Winter',
  suprimento: 'Post-its',
  quantidade: 5000,
  custo_unitario: Decimal128('852.65'),
  custo_total: Decimal128('4263250.00')
}
escritorio: "Shuri's Palace",
  suprimento: 'Post-its',
  quantidade: 5000,
  custo_unitario: Decimal128('1000.00'),
  custo_total: Decimal128('5000000.00')
    escritorio: 'Baxter Building',
  suprimento: 'Papel',
  quantidade: 5000,
  custo_unitario: Decimal128('752.85'),
  custo_total: Decimal128('3764250.00')
    escritorio: 'Baxter Building',
  suprimento: 'Planners',
  quantidade: 2010,
  custo_unitario: Decimal128('500.00'),
  custo_total: Decimal128('1005000.00')
   escritorio: 'Momento Brasil',
  suprimento: 'Papel A4',
  quantidade: 100,
  custo_unitario: Decimal128('32.50'),
  custo_total: Decimal128('3250.00')
   escritorio: 'Momento Brasil',
  suprimento: 'Caneta Azul',
  quantidade: 80,
  custo_unitario: Decimal128('2.50'),
  custo_total: Decimal128('200.00')

db.escritorio.aggregate([
  {
    $unwind: "$suprimentos"
  },
  {
    $match: {
      "suprimentos.quantidade": { $gt: 50 }
    }
  },
  {
    $project: {
      _id: 0,
      escritorio: "$nome",
      suprimento: "$suprimentos.nome",
      quantidade: "$suprimentos.quantidade",
      custo_unitario: "$suprimentos.custo",
      custo_total: {
        $multiply: [
          "$suprimentos.quantidade",
          "$suprimentos.custo"
        ]
      }
    }
  },
  {
    $sort: {
      quantidade: -1
    }
  }
])

10.6 Calcule quanto a empresa economizaria se reduzisse 20% dos salários acima de $15.000 (apenas calcule, não execute).
R: folha_salarial_atual: Decimal128('144930.00'),
  quantidade_funcionarios: 7,
  economia_20_por_cento: Decimal128('28986.00000000000000000'),
  folha_apos_reducao: Decimal128('115944.00000000000000000')

db.funcionarios.aggregate([
  {
    $match: {
      salario: { $gt: 15000 }
    }
  },
  {
    $group: {
      _id: null,
      folha_salarial_atual: {
        $sum: "$salario"
      },
      quantidade_funcionarios: {
        $sum: 1
      },
      economia_20_por_cento: {
        $sum: {
          $multiply: [
            "$salario",
            0.20
          ]
        }
      }
    }
  },
  {
    $project: {
      _id: 0,
      quantidade_funcionarios: 1,
      folha_salarial_atual: 1,
      economia_20_por_cento: 1,
      folha_apos_reducao: {
        $subtract: [
          "$folha_salarial_atual",
          "$economia_20_por_cento"
        ]
      }
    }
  }
])

# Cenário 3: Auditoria de Vendas
O departamento financeiro suspeita de irregularidades.

10.7 Encontre vendas que não possuem vendedor associado.
R:
db.vendas.aggregate([
  {
    $lookup: {
      from: "funcionarios",
      localField: "funcionario_id",
      foreignField: "_id",
      as: "funcionario"
    }
  },
  {
    $match: {
      $or: [
        { funcionario_id: { $exists: false } },
        { funcionario_id: null },
        { funcionario: { $size: 0 } }
      ]
    }
  }
])

10.8 Verifique se há vendedores registrados nas vendas que não existem na coleção de funcionários.
R:
db.vendas.aggregate([
  {
    $lookup: {
      from: "funcionarios",
      localField: "funcionario_id",
      foreignField: "_id",
      as: "funcionario"
    }
  },
  {
    $match: {
      funcionario_id: { $ne: null },
      funcionario: { $size: 0 }
    }
  },
  {
    $project: {
      _id: 1,
      produto_id: 1,
      quantidade: 1,
      data_venda: 1,
      funcionario_id: 1
    }
  }
])

10.9 Identifique produtos que aparecem nas vendas mas com preços unitários muito diferentes entre transações (variação > 10%).
R:
  _id: 3900,
  nome: 'Baxter Building',
  custo_total_suprimentos: Decimal128('20839750.00')
   _id: 2500,
  nome: 'Umbrella Corp',
  custo_total_suprimentos: Decimal128('6155000.00')

 _id: 2400,
  nome: "Shuri's Palace",
  custo_total_suprimentos: Decimal128('5147785.00')

db.vendas.aggregate([
  {
    $match: {
      preco_unitario: { $gt: 0 }
    }
  },
  {
    $group: {
      _id: "$produto_id",
      menor_preco: { $min: "$preco_unitario" },
      maior_preco: { $max: "$preco_unitario" }
    }
  },
  {
    $match: {
      $expr: {
        $gt: [
          {
            $divide: [
              {
                $subtract: [
                  "$maior_preco",
                  "$menor_preco"
                ]
              },
              "$menor_preco"
            ]
          },
          0.10
        ]
      }
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
      _id: 0,
      produto: "$produto.nome",
      menor_preco: 1,
      maior_preco: 1,
      variacao_percentual: {
        $multiply: [
          {
            $divide: [
              {
                $subtract: [
                  "$maior_preco",
                  "$menor_preco"
                ]
              },
              "$menor_preco"
            ]
          },
          100
        ]
      }
    }
  }
])


# Cenário 4: Relatório Executivo
O CEO pediu um dashboard completo.

10.10 Crie UMA ÚNICA query de agregação que retorne:

Total de funcionários
Custo total com salários
Número de departamentos
Número de escritórios
Receita total de vendas
Produto mais vendido

R:
  total_funcionarios: 52,
  custo_total_salarios: Decimal128('478560.0100'),
  total_departamentos: 15,
  total_escritorios: 10,
  receita_total_vendas: Decimal128('159186.48'),
  produto_mais_vendido: {
    quantidade_vendida: 177,
    produto: 'Uniforme do Superman'
db.funcionarios.aggregate([
  {
    $facet: {
      resumo_funcionarios: [
        {
          $group: {
            _id: null,
            total_funcionarios: { $sum: 1 },
            custo_total_salarios: { $sum: "$salario" }
          }
        },
        {
          $project: {
            _id: 0,
            total_funcionarios: 1,
            custo_total_salarios: 1
          }
        }
      ],

      resumo_departamentos: [
        { $limit: 1 },
        {
          $lookup: {
            from: "departamentos",
            pipeline: [{ $count: "total" }],
            as: "contagem"
          }
        },
        {
          $project: {
            _id: 0,
            total_departamentos: {
              $ifNull: [
                { $arrayElemAt: ["$contagem.total", 0] },
                0
              ]
            }
          }
        }
      ],

      resumo_escritorios: [
        { $limit: 1 },
        {
          $lookup: {
            from: "escritorio",
            pipeline: [{ $count: "total" }],
            as: "contagem"
          }
        },
        {
          $project: {
            _id: 0,
            total_escritorios: {
              $ifNull: [
                { $arrayElemAt: ["$contagem.total", 0] },
                0
              ]
            }
          }
        }
      ],

      resumo_vendas: [
        { $limit: 1 },
        {
          $lookup: {
            from: "vendas",
            pipeline: [
              {
                $lookup: {
                  from: "produtos",
                  localField: "produto_id",
                  foreignField: "_id",
                  as: "produto"
                }
              },
              { $unwind: "$produto" },
              {
                $group: {
                  _id: null,
                  receita_total: {
                    $sum: {
                      $multiply: [
                        "$quantidade",
                        "$produto.preco_unitario"
                      ]
                    }
                  }
                }
              }
            ],
            as: "dados"
          }
        },
        {
          $project: {
            _id: 0,
            receita_total: {
              $ifNull: [
                { $arrayElemAt: ["$dados.receita_total", 0] },
                0
              ]
            }
          }
        }
      ],

      produto_mais_vendido: [
        { $limit: 1 },
        {
          $lookup: {
            from: "vendas",
            pipeline: [
              {
                $group: {
                  _id: "$produto_id",
                  quantidade_vendida: { $sum: "$quantidade" }
                }
              },
              { $sort: { quantidade_vendida: -1 } },
              { $limit: 1 },
              {
                $lookup: {
                  from: "produtos",
                  localField: "_id",
                  foreignField: "_id",
                  as: "produto"
                }
              },
              { $unwind: "$produto" },
              {
                $project: {
                  _id: 0,
                  produto: "$produto.nome",
                  quantidade_vendida: 1
                }
              }
            ],
            as: "mais_vendido"
          }
        },
        {
          $project: {
            _id: 0,
            produto_mais_vendido: {
              $arrayElemAt: ["$mais_vendido", 0]
            }
          }
        }
      ]
    }
  },
  {
    $project: {
      _id: 0,
      total_funcionarios: {
        $arrayElemAt: ["$resumo_funcionarios.total_funcionarios", 0]
      },
      custo_total_salarios: {
        $arrayElemAt: ["$resumo_funcionarios.custo_total_salarios", 0]
      },
      total_departamentos: {
        $arrayElemAt: ["$resumo_departamentos.total_departamentos", 0]
      },
      total_escritorios: {
        $arrayElemAt: ["$resumo_escritorios.total_escritorios", 0]
      },
      receita_total_vendas: {
        $arrayElemAt: ["$resumo_vendas.receita_total", 0]
      },
      produto_mais_vendido: {
        $arrayElemAt: ["$produto_mais_vendido.produto_mais_vendido", 0]
      }
    }
  }
])
