---
title: "[TCC] Proposta do Projeto"
date: 2026-04-18 16:15:00 +/-0300
categories: [TCC, Multimodal]
description: "Esse post cobre a proposta do projeto de conclusão de curso. Exclusivamente, esse post será feito em português."
tags: [machine learning, AI, ML, IQA, retina, LLM, captioning, multimodal]
math: true
---

[Título]:
- Bridging Text and Retinal Imaging: A Multimodal Search Approach
- Multimodal Embeddings for Text-Image Retrieval in Retinal Image Databases
- Semantic Alignment between Text and Retinal Images for Medical Retrieval Systems
- Natural Language Retrieval of Retinal Images for Clinical Decision Support Systems
- Retinal Images Retrieval Based on Medical Semantic Queries: A Multimodal Approach
- Multimodal Search in Retinal Images Databases based on Natural Language Processing

Rodrigo de Castro Michelassi\\
Orientadora: Nina S. T. Hirata

### Introdução 

Imagens de retina são uma rica fonte de informações relevantes para diagnosticar doenças e anomalias. Além de possibilitar a manifestação de diversas doenças que se manifestam diretamente na retina e nos vasos sanguíneos, essas imagens também possuem grande importância no campo da medicina por serem dados não invasivos, que podem ser coletados de maneira barata, por meio de retinógrafos portáteis, e enriquecerem bases para pesquisa.

No campo de Aprendizado de Máquina, o trabalho com imagens de retina tem sido amplamente explorado, com aplicações focadas em detectar e segmentar suas estruturas anatômicas, como o disco óptico, fóvea e vasos sanguíneos, treinar modelos para classificação de doenças, avaliar a qualidade das imagens, entre outros. Nesse contexto, pretendemos expandir ainda mais o arcabouço científico que engloba a exploração desse tipo de dados, na área de Aprendizado de Máquina, provendo ferramentas para auxiliar profissionais da oftalmologia a trabalharem com imagens de retina e tomarem decisões mais acuradas e eficientes durante o atendimento de pacientes. 

Nossa proposta consiste em treinar modelos de alinhamento texto-imagem, como CLIP e SigLIP, permitindo a criação de uma base de dados de imagens de retina e um sistema de busca nessa base de dados com linguagem natural. Dessa forma, dado que um profissional de oftalmologia deseje analisar imagens que, por exemplo, o paciente possua retinopatia diabética, basta fazer uma busca por “retinopatia diabética”, e o sistema poderá filtrar todas as imagens que se enquadrem nessa descrição. O mesmo deve valer para descrições mais complexas, feitas em linguagem natural.

Esse problema, além de trazer benefícios para profissionais de medicina, ao criar uma base de dados de retina de fácil busca, também possibilita a expansão de projetos na área. Como uma possível continuação para esse projeto, pretendemos utilizar o modelo já treinado de alinhamento texto-imagem, especificamente o _image encoder_, para treinar um outro modelo de aprendizado de máquina, com uma arquitetura de _vision language model_ (VLM), capaz de gerar legendas com descrições de imagens de retina, trazendo praticidade para profissionais, que poderão analisar imagens com um conhecimento prévio de qual a situação apresentada na retina do paciente. Essa proposta se enquadra como um projeto extra a ser abordado, no contexto de um trabalho de conclusão de curso. 

Assim, com esse projeto, pretendemos ampliar a disponibilidade do conhecimento no campo da oftalmologia, fornecendo ferramentas modernas para profissionais da área, que auxiliem na disseminação do conhecimento e na modernização de centros de atendimento médico. Além disso, esse projeto atua diretamente na democratização de acesso a ferramentas de qualidade em locais com pouca ou nenhuma disponibilidade de profissionais oftalmologistas, fato recorrente em diversas regiões do Brasil.

### Background e Fundamentos

No passado, tivemos a oportunidade de participar de alguns projetos envolvendo imagens de retina. Esses projetos vieram de uma parceria entre o Instituto de Matemática, Estatística e Ciência da Computação (IME-USP) e a Faculdade de Medicina (FMUSP) da Universidade de São Paulo, com o objetivo primário de investigar causas para declínio cognitivo em pacientes. Uma das principais fontes de informação e dados utilizados para isso são, de fato, as imagens de retina. Como um projeto secundário da FMUSP, surge a instalação de retinógrafos portáteis como ferramentas de triagem médica, dado que esses são mais baratos que retinógrafos profissionais e as imagens podem ser capturadas pela câmera do próprio celular, possibilitando que, dado a comprovação da eficiência do projeto, pode resultar em um investimento na saúde em regiões em que a presença de oftalmologistas é escassa. 

No contexto apresentado, desenvolvemos dois projetos de aprendizado de máquina com imagens de retina. O primeiro projeto consistiu em treinar um modelo de detecção de objetos (YOLO) para detectar estruturas como disco óptico e mácula em imagens de retina. Com base na localização dessas estruturas, criamos um algoritmo para definir a posição que eles se encontram na imagem e, geometricamente, definir se a imagem possui qualidade adequada para verificar a presença de retinopatia diabética, seguindo um protocolo médico conhecido. Já no segundo projeto, treinamos 14 modelos de Few-Shot Learning, um paradigma de Aprendizado de Máquina que permite treinar modelos com pouca disponibilidade de dados, que são fortemente adaptativos com novas classes não vistas, a fim de classificar imagens de retina com base em um seleto grupo de doenças apresentadas no dataset BRSet. 

Nesse projeto, temos como objetivo dar continuidade nesse grupo de ferramentas que desenvolvemos para diversos contextos de imagens de retina. Durante os projetos anteriores, o foco de desenvolvimento se deu inteiramente sobre uma única fonte de informação: imagens. Agora, pretendemos expandir o leque de opções e enriquecer ainda mais a pesquisa nesse campo, utilizando modelos multimodais, ou seja, que possam ser treinados em cima de vários tipos de dados. A princípio nosso objetivo é introduzir dados textuais na pesquisa.

Assim, define-se como nosso escopo inicial do projeto, treinar um modelo de alinhamento texto-imagem, com o qual podemos mapear embeddings de ambos os tipos de dados no mesmo espaço vetorial. Esse tipo de modelo é útil pois possibilita que diferentes fontes de dados, porém que possuem um mesmo significado semântico, sejam representados de maneiras parecidas (por exemplo: uma imagem que representa um cachorro seria mapeada para a mesma região do espaço que um texto descrevendo um cachorro). No contexto do projeto de imagens de retina, nosso objetivo seria alinhar imagens de retina com suas respectivas descrições textuais, possibilitando encontrar imagens com determinadas características mais facilmente. A partir disso, pretendemos criar uma ferramenta de busca baseada em linguagem natural, capaz de buscar imagens de retina com determinada característica, armazenadas em uma base de dados. 

Esse problema é motivador para dar segmento ao desenvolvimento multimodal na área da oftalmologia, em que pretendemos, no futuro, treinar vision-language models, capazes de automaticamente gerar legendas descritivas para imagens de retina, que podem ser capazes de auxiliar profissionais da área a analisar informações corretamente após exames realizados com um retinógrafo. Esse projeto não necessariamente faz parte do escopo desse trabalho de conclusão de curso.

### Metodologia

Para definir esse projeto, seguiremos uma sequência de passos lógicos utilizados no campo de Aprendizado de Máquina. Como um primeiro passo, precisamos revisar a literatura, verificar a disponibilidade de dados para treinar os modelos que precisamos e alinhar as tecnologias que pretendemos usar.

Primeiramente, nosso objetivo é reutilizar datasets que já exploramos anteriormente, como o BRSet, mencionado na seção anterior, e expandir os horizontes para outros datasets públicos de imagens de retina, que contenham classificação de doenças. Todavia, no geral existem poucos datasets públicos com anotações textuais que descrevem imagens de retina, e a tarefa se torna ainda mais difícil quando pensamos em datasets cujas descrições em linguagem natural se alinhem com nosso objetivo. 

Com a finalidade de termos anotações que se alinhem com nossas expectativas, pretendemos gerar nossas próprias anotações para o desenvolvimento da ferramenta de busca. Utilizaremos, como fonte de informação, as anotações já existentes nos datasets, como as labels de classificação, juntamente ao nosso modelo já treinado de detecção de objetos, que possibilita descrever o lado de cada objeto na imagem, e alimentar essas informações a um Large Language Model (LLM), com um prompt adequado para gerar descrições textuais para cada imagem. Como há grande disponibilidade de imagens, faremos uma análise das anotações obtidas em cima de uma amostra de tamanho adequado. 

Tendo os dados anotados, pretendemos iniciar com o treinamento de dois modelos de alinhamento texto-imagem:

- CLIP: Modelo de 2021, lançado pela OpenAI. Um dos modelos mais famosos do segmento.
- SigLIP: Modelo de 2023, lançado pela Google Deepmind. Esse modelo vem como um dos sucessores do CLIP.

Nosso objetivo é revisar a literatura em torno desses dois modelos, entender como são treinados e como podemos organizar e utilizar nossos dados. A partir disso, poderemos definir quais métricas são passíveis de serem avaliadas para esse tipo de modelo multimodal e o que elas representam, e fazer uma comparação sobre a performance de ambos os modelos.

Por fim, seguiremos para um trabalho de engenharia, para entender de que forma podemos construir um sistema de busca em uma base de dados de imagens, com base em informações de linguagem natural, utilizando o modelo treinado que obteve o melhor resultado em nossas comparações. Tendo seguido todos esses passos, teremos atingido os objetivos iniciais definidos previamente, na seção de background.

### Plano de Atividades

Abaixo, segue o plano de atividades, com todas ações que precisamos tomar a fim de
atingir os objetivos propostos, não necessariamente em ordem cronológica:

- [ ] Estudar a literatura em torno de modelos multimodais para alinhamento texto-imagem.
    - [ ] CLIP.
    - [ ] SigLIP.
- [ ] Estudar a literatura do uso desses modelos em imagens médicas.
    - [ ] Como são usados atualmente na literatura.
    - [ ] Que métricas são usadas para avaliar a performance dos modelos.
    - [ ] Quais os objetivos de aplicação desses modelos.
- [ ] Adaptação do dataset para treinamento.
- [ ] Análise do dataset gerado.
- [ ] Anotação de métricas de avaliação para os modelos.
- [ ] Treinamento e avaliação do CLIP.
- [ ] Treinamento e avaliação do SigLIP.
- [ ] Análise e comparação dos resultados.
- [ ] Organização do código desenvolvido para treinamento dos modelos.
    - [ ] Treinamento baseado em seeds pré-definidas para reprodução de resultados.
    - [ ] Arquitetura escalável para sistemas de treinamento de modelos de Aprendizado de Máquina.
- [ ] Desenvolvimento da ferramenta para busca em base de dados de imagens.
- [ ] Organização do código de desenvolvimento para a busca da base de dados.
- [ ] Disponibilização em código livre dos códigos utilizados.
- [ ] (Bônus): seguir o mesmo padrão de desenvolvimento para o treinamento de um VLM para geração de legendas descritivas de imagens de retina, se condizente com o prazo de entrega do projeto

### Bibliografia Sugerida

Abaixo, apresentamos uma bibliografia inicial sugerida para o projeto. Essas fontes serão revisadas, com alguns textos possivelmente não sendo utilizados e outros sendo adicionados.

[1] E. Trucco, T. MacGillivray, and Y. Xu, Eds., Computational Retinal
Image Analysis – Tools, Applications and Perspectives, ser. The Elsevier
and MICCAI Society Book Series. Academic Press, 2019.

[2] J. Redmon, S. Divvala, R. Girshick, and A. Farhadi, “You Only Look
Once: Unified, Real-Time Object Detection,” in IEEE Conference on
Computer Vision and Pattern Recognition (CVPR). Los Alamitos, CA,
USA: IEEE Computer Society, 2016, pp. 779–788.

[3] L. F. Nakayama, D. Restrepo, J. Matos, L. Z. Ribeiro, F. K. Malerbi,
L. A. Celi, and C. S. Regatieri, “BRSET: A Brazilian Multilabel
Ophthalmological Dataset of Retina Fundus Photos,” PLOS Digital
Health, vol. 3, no. 7, pp. 1–16, 07 2024. [Online]. Available:
https://doi.org/10.1371/journal.pdig.0000454

[4] G. Jocher and J. Qiu, “Ultralytics YOLO11,” 2024. [Online]. Available:
https://github.com/ultralytics/ultralytics

[5] R. Michelassi and N. Hirata, “Detection of retinal anatomical structures
and its application to image quality assessment,” in Anais Estendidos
da XXXVIII Conference on Graphics, Patterns and Images. Porto
Alegre, RS, Brasil: SBC, 2025, pp. 259–262. [Online]. Available:
https://sol.sbc.org.br/index.php/sibgrapi estendido/article/view/38309

[6] A. Radford, J. W. Kim, C. Hallacy, A. Ramesh, G. Goh, S. Agarwal,
G. Sastry, A. Askell, P. Mishkin, J. Clark, G. Krueger, and
I. Sutskever, “Learning transferable visual models from natural
language supervision,” CoRR, vol. abs/2103.00020, 2021. [Online].
Available: https://arxiv.org/abs/2103.00020

[7] X. Zhai, B. Mustafa, A. Kolesnikov, and L. Beyer, “Sigmoid loss
for language image pre-training,” in Proceedings of the IEEE/CVF
International Conference on Computer Vision (ICCV), 2023.

[8] M. Sultan, L. Jacobs, A. Stylianou, and R. Pless, “Exploring clip for
real world, text-based image retrieval,” in 2023 IEEE Applied Imagery
Pattern Recognition Workshop (AIPR), 2023, pp. 1–6.

[9] K. Schall, K. U. Barthel, N. Hezel, and K. Jung, “Optimizing clip
models for image retrieval with maintained joint-embedding alignment,” 2024. 
[Online]. Available: https://arxiv.org/abs/2409.01936

[10] F. Faghri, D. J. Fleet, J. R. Kiros, and S. Fidler, “VSE++:
improved visual-semantic embeddings,” CoRR, vol. abs/1707.05612, 2017. 
[Online]. Available: http://arxiv.org/abs/1707.05612




> This post is used as a bachelor thesis checkpoint for the image captioning on retinal fundus images, supervised by professor [Nina S. T. Hirata](https://www.ime.usp.br/~nina/), from the Institute of Mathematics, Statistics and Computer Science at the University of São Paulo (IME USP).
{: .prompt-info }