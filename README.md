# Sorteador UFSCar

**URL da ferramenta: <https://sorteador.ufscar.br>**

O **Sorteador UFSCar** é uma ferramenta de sorteio auditável desenvolvida para trazer mais transparência, segurança e praticidade aos sorteios em concursos públicos, processos seletivos e outras necessidades da Universidade Federal de São Carlos (UFSCar) e da comunidade em geral.

Este projeto foi criado para superar os desafios de métodos de sorteio tradicionais, como sites genéricos que não permitem auditoria ou a Loteria Federal, que possui limitações de frequência (apenas duas vezes por semana) e de escopo numérico (números de 5 dígitos, insuficientes para sorteios de permutação complexos).

### Vídeo de Apresentação

Para uma demonstração visual completa do seu funcionamento, auditabilidade e casos de uso, assista ao vídeo de apresentação:

<a href="https://www.youtube.com/watch?v=ZBwKWcE_B9k" target="_blank"><img src="https://img.youtube.com/vi/ZBwKWcE_B9k/mqdefault.jpg" alt="Vídeo de Apresentação do Sorteador UFSCar" width="500" /></a>

## Principais Características

*   **Auditabilidade Total:** Qualquer pessoa pode verificar o resultado de um sorteio passado, usando os mesmos parâmetros de entrada, e obterá o mesmo resultado e o mesmo log de execução.
*   **Transparência Criptográfica:** A ferramenta não gera seus próprios números "aleatórios". Em vez disso, ela utiliza uma fonte de aleatoriedade pública e descentralizada, a rede **[drand](https://drand.love)**.
*   **Flexibilidade de Agendamento:** Os sorteios são vinculados a um horário específico (com precisão de minuto). Isso permite agendar sorteios para qualquer momento no futuro, com total flexibilidade.
*   **Resultados Reprodutíveis e Incontestáveis:** O processo de derivação do resultado a partir da fonte de aleatoriedade pública é determinístico e transparente. Ele combina a aleatoriedade da `drand` com os dados públicos do sorteio (como a identificação do edital) usando um algoritmo de hash padrão (SHA3-224).
*   **Operação Descentralizada:** A fonte de aleatoriedade é mantida por um consórcio global de universidades e empresas, garantindo que não haja um ponto único de falha ou manipulação. A ferramenta da UFSCar é uma interface segura para consumir esses dados.
*   **Independência de Sorteios:** O campo "Identificação do Edital" atua como um "sal" criptográfico, garantindo que dois sorteios diferentes (ex: de departamentos diferentes) realizados com os mesmos parâmetros de quantidade no mesmo horário gerem resultados completamente independentes.

## Como Funciona

A ferramenta opera inteiramente no lado do cliente (no seu navegador), seguindo um processo claro e verificável:

1.  **Entrada de Dados:** O usuário fornece os parâmetros do sorteio:
    *   `Identificação do Edital`: Um texto único para o sorteio.
    *   `Horário do Sorteio`: A data e hora exatas para a realização do sorteio.
    *   `Quantidade`: Quantos números sortear.
    *   `Maior valor`: O limite superior do intervalo do sorteio (de 1 a N).

2.  **Vinculação com a Rede drand:** A ferramenta calcula o número do "round" da rede `drand` que corresponde exatamente ao `Horário do Sorteio` configurado. A `drand` publica um novo número aleatório único a cada 30 segundos, e cada publicação tem um número de `round` sequencial.

3.  **Geração do Resultado:**
    *   Quando o `round` agendado se torna público, a ferramenta obtém o valor aleatório de 256 bits (uma "semente") correspondente.
    *   Ela então cria uma string combinando a `Identificação do Edital`, a semente da `drand` e um contador de tentativas.
    *   Essa string é processada pela função de hash criptográfica **SHA3-224**.
    *   O resultado do hash é convertido em um número dentro do intervalo desejado. É utilizada uma técnica chamada **rejeição de amostragem** (*rejection sampling*) para garantir que todos os números no intervalo tenham a mesma probabilidade de serem sorteados, evitando qualquer viés.
    *   Para sorteios múltiplos (sequências), o processo é repetido, descartando números que já saíram, para garantir que não haja repetições.

4.  **Log de Transparência:** Todo o processo de derivação, incluindo cada tentativa de hash, é exibido em um log detalhado, permitindo a qualquer pessoa entender como o resultado final foi alcançado.

## Uso da Ferramenta

### 1. Realizando um Sorteio

Para realizar um sorteio que ainda não aconteceu.

1.  **Preencha os campos:** Informe a identificação, o horário (que deve ser no futuro), a quantidade de números e o maior valor possível.
2.  **Clique em "Sortear"**: A ferramenta irá aguardar até o horário configurado.
3.  **Aguarde o Resultado**: No momento exato, a ferramenta buscará a aleatoriedade na rede `drand` e exibirá o resultado e o log de execução.

**Exemplo de Log:**
```
Resultado do sorteio
6

round = 5121016
(sha3_224('00125.01|835efe4e4ba94a4388cf3c32e3a0d794d1493ed8890d999dca06870207bdf9fb|0') & 0xf) + 1 = 13
(sha3_224('00125.01|835efe4e4ba94a4388cf3c32e3a0d794d1493ed8890d999dca06870207bdf9fb|1') & 0xf) + 1 = 11
(sha3_224('00125.01|835efe4e4ba94a4388cf3c32e3a0d794d1493ed8890d999dca06870207bdf9fb|2') & 0xf) + 1 = 6
```
Neste exemplo, as duas primeiras tentativas geraram números fora do intervalo desejado (maiores que 10) e foram descartadas, até que a terceira tentativa produziu um resultado válido.

### 2. Auditando um Sorteio

Para verificar um sorteio que já aconteceu.

1.  **Preencha os campos com os dados originais:** É crucial que **todos** os campos (`Identificação do Edital`, `Horário do Sorteio`, `Quantidade` e `Maior valor`) sejam **exatamente os mesmos** do sorteio original.
2.  **Clique em "Auditar"**: A ferramenta não vai esperar. Ela buscará o `round` da `drand` que já é público e corresponde ao horário informado.
3.  **Confirme o Resultado**: O resultado e o log exibidos serão idênticos aos do sorteio original, confirmando a sua lisura.

### 3. Sorteios Agendados (Sem Operação ao Vivo)

É possível definir os parâmetros de um sorteio publicamente (por exemplo, em um edital de concurso) e não realizar a operação "ao vivo".

Como a rede `drand` é autônoma, o `round` correspondente ao horário agendado será gerado e se tornará público independentemente de alguém acessar o site do Sorteador. Posteriormente, qualquer pessoa poderá usar a função **Auditar**, com os dados publicados no edital, para obter o resultado oficial.

> **Atenção:** Para esta modalidade, é fundamental que **todos os parâmetros** do formulário sejam publicados de forma precisa e inequívoca no documento oficial. Qualquer erro ou omissão pode inviabilizar a auditoria ou abrir margem para questionamentos.

## Código-Fonte e Tecnologias

Este é um projeto *open-source* e seu código-fonte está disponível neste repositório.

*   **Estrutura:** A ferramenta é uma aplicação de página única (*Single-Page Application*) totalmente executada no lado do cliente. Não há um backend no servidor da UFSCar para a lógica do sorteio, o que reforça a transparência.
*   **Tecnologias Principais:**
    *   **HTML/CSS/JavaScript:** A base da aplicação web.
    *   **Bootstrap 5:** Para a interface e responsividade.
    *   **Tempus Dominus:** Componente de seleção de data e hora.
    *   **drand-client:** Biblioteca JavaScript para interagir com a rede `drand`.
    *   **js-sha3:** Implementação da função de hash SHA3-224.
