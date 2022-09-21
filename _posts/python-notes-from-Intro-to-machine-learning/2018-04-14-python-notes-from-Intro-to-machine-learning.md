---
title: Terraform Básico #1
date: 2022-07-09 10:00:00 +07:00
tags: [terraform, devops]
description: Explicando o básico do Terraform.
image: /cara-memperbarui-fork-repository/repo.png
---

### O que é o Terraform?

Terraform é uma ferramenta IAC open source criada pela [Hashicorp](https://www.hashicorp.com/) que utiliza a linguagem HCL(derivada do YAML) para o provisionamento de infraestruturas em diversas plataformas cloud.

### Arquitetura

O Terraform não faz uma comunicação direta com a plataforma, ele faz o uso de Providers que são plugins e esses fazem essa ponte com a cloud

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 1. Arquitetura Terraform.</figcaption>
</figure>


### Como ele funciona?

O binário do terraform lê o arquivo HCL(o seu codigo.tf), publica na cloud e após a publicação é armazeando o estado do que foi feito no arquivo de state.

### Primeiros passos:

De início baixar o binário do terraform no [site oficial](https://www.terraform.io/downloads) se você estiver usando Windows, caso esteja no Linux ou MacOs utilizar a linha de comando no terminal mesmo.

- `apt install terraform(Linux)`
- `brew install terraform(Mac)`

Também podemos criar um container com o docker contendo a imagem do terraform, caso prefira dessa forma, aqui está o [link](https://hub.docker.com/r/hashicorp/terraform/) da imagem

### Plataforma Cloud:

Nesse caso escolhi a AWS para o provisionamento da infraestrutura, nela mesmo criei um usuário denominado terraform e apliquei uma role com a policy de administrator apenas para fazer o teste da ferramenta.

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 2. Users - AWS.</figcaption>
</figure>

Após a criação do usuário gerei as credenciais AWS Access Key e Secret Acces Key, pois essas duas chaves são necessárias para o Terraform saber quem é o usuário que irá provisionar os recursos.

Criei um bucket s3 que irá servir para armazenar nosso arquivo de state do Terraform, mas você pode também deixar esse arquivo local na máquina.

### Exportando as chaves:

Abra o terminal do seu SO e exporte as chaves geradas do seu usuário terraform, como estou no windows, fiz da seguinte forma no Power Shell:

- `Set-Item -Path env:AWS_ACCESS_KEY_ID -Value "valor"`
- `Set-Item -Path env:AWS_SECRET_ACCESS_KEY -Value "valor"`

Por questões de seguranç  não coloquei as chaves de acesso da AWS no código .tf apenas exportei como variáveis de ambiente.

Uma boa prática é você colocar essas chaves em um cofre como o Vault da própria Hashicorp.

### Iniciando o Terraform: 

Com tudo preparado, utilizei os comandos abaixo para iniciar o terraform:

- **terraform init =** Inicia o terraform criando o arquivo .terraform na pasta, faz a leitura de todos os arquivos que contém a extensão .tf no nome.
- **terraform init -upgrade =** Faz as mesmas ações do outro comando, porém atualiza também todos os plugins que você tem

**Pasta .terraform =** Contém tudo que o binário local precisa para realizar as execuções.

Eu prefiro sempre iniciar com o que tem a flag de upgrade, pois caso tenha algo desatualizado ele ja atualiza, resultado do comando:

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 3. Terraform init.</figcaption>
</figure>

Após iniciado, vamos criar uma máquina lá na AWS utilizando o código **main.tf** e **ec2.tf** descritos abaixo:

#### **main.tf**: 
```terraform
provider "aws" {
    region = "us-east-1"
    version = "~> 2.7"
}

terraform {
    backend "s3" {
        bucket = "nomedoseubucket"
        key = "terraform-test-2.tfstate"
        region = "us-east-1"
    }
}
```


#### **ec2.tf**:
```terraform
resource "aws_instance" "mwtt4" {
    ami = "ami-0b0ea68c435eb488d"
    instance_type = "t2.micro"

tags = {
    Name = "Hello"
 }
} 
```

Para isso podemos utilizar o comando **terraform plan** que verifica tudo que está no código e te da um panorama do que vai ser feito, faz um diff entre a infraestrutura (máquinas que você já tem) o state e faz a criação.

Existe também o comando **terraform plan -out infra-1** que envia o plano para um arquivo binário(nesse caso será o infra-1) assim deixando mais organizado suas alterações futuras.

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 4. terraform plan.</figcaption>
</figure>

- o **+** indica que a máquina e os recursos vão ser criados

Agora vamos utilizar o comando **terraform apply "infra-1"** para criar de fato tudo que apareceu na imagem anterior:

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 5. terraform apply.</figcaption>
</figure>

A criação foi feita com sucesso, validando no console da aws:

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 6. AWS Result.</figcaption>
</figure>

Validando o arquivo de state no bucket:

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 7. Bucket.</figcaption>
</figure>

Após criado vamos apagar tudo, para isso utilizamos o comando **terraform destroy**:

<figure>
<img src="{{ page.image }}" alt="ilustrasi repo yang mau diupdate">
<figcaption>Fig 8. terraform destroy.</figcaption>
</figure>

- o **-** indica que a máquina e os recursos vão ser criados

Feito, muito mais fácil do que passar item por item no console e ir clicando, sem falar que da forma manual a chance de errar é bem grande, no código fica tudo mais prático e padronizado.

Com isso, finalizo aqui essa primeira parte, existem muitas outras funcionalidades do Terraform, mas vou deixar para outros posts.
