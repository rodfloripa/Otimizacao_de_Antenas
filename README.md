
---

<p align="justify"><h1>Projeto de Antenas com CVXPY</h1></p>

<p align="justify">Projetar uma antena usando o <b>CVXPY</b> é, essencialmente, transformar um problema de eletromagnetismo em um problema de <b>otimização matemática</b>. Em vez de tentar adivinhar os valores de amplitude e fase para cada elemento da antena, você define o "comportamento desejado" (o lóbulo) e deixa que o algoritmo encontre a melhor configuração física para alcançá-lo. Abaixo, detalhamos o funcionamento desse processo:</p>

<p align="justify"><h3>1. O Conceito: A Antena como uma Soma Ponderada</h3></p>

<p align="justify">Imagine que você tem vários pequenos alto-falantes (elementos da antena) em linha. Se todos tocarem o mesmo som ao mesmo tempo, o som será mais forte bem na frente deles. Se você atrasar o som de alguns, o "foco" do som vira para o lado. Matematicamente, o campo total irradiado <b>A(θ)</b> é a soma das contribuições de cada elemento <b>n</b>, multiplicada por um peso complexo <b>w<sub>n</sub></b>:</p>

<p align="center">
<b>A(θ) = Σ w<sub>n</sub> · exp(j · ϕ<sub>n</sub>(θ))</b>
</p>

<p align="justify">No <b>CVXPY</b>, o nosso objetivo é encontrar o vetor de pesos <b>w</b> que molda essa soma para criar o foco de energia desejado.</p>

<p align="justify"><h3>2. A "Mágica" da Convexidade</h3></p>

<p align="justify">O grande trunfo de usar otimização convexa (CVX) em antenas é que muitos requisitos de projeto podem ser escritos como restrições lineares ou quadráticas:</p>

* <p align="justify"><b>Direção do Feixe:</b> "Eu quero que o ganho em 0° seja exatamente 1." (Restrição de igualdade).</p>
* <p align="justify"><b>Minimização de Ruído (Side Lobes):</b> "Eu quero que o ganho nos lóbulos secundários seja o menor possível." (Minimização da norma infinita).</p>
* <p align="justify"><b>Anulação de Interferência (Null Steering):</b> "Eu quero que o ganho em -30° seja zero." (Restrição para ignorar sinais indesejados).</p>

<p align="justify"><h3>3. Restrição de Ganho Unitário: O "Foco" da Antena</h3></p>

<p align="justify">A linha de código <code>constraints = [A_mat[np.argmin(np.abs(theta - target_angle))] @ w == 1]</code> é o mecanismo técnico que define o direcionamento do feixe. Ela funciona da seguinte forma:</p>

<p align="justify">1. <b>Identificação do Alvo:</b> O trecho <code>np.argmin(np.abs(theta - target_angle))</code> funciona como uma mira telescópica. Como o vetor <code>theta</code> contém milhares de ângulos possíveis, essa função encontra o índice exato da linha na matriz <b>A</b> que representa a física da direção desejada.</p>

<p align="justify">2. <b>Resposta do Sistema:</b> O produto matricial <code>A_mat[...] @ w</code> calcula a soma vetorial de todos os sinais emitidos. Matematicamente, cada elemento contribui com um sinal cuja fase depende da posição; os pesos <b>w</b> ajustam esses sinais para que cheguem ao destino em fase.</p>

<p align="center">
<b>Sinal Total = w<sub>0</sub>e<sup>jϕ<sub>0</sub></sup> + w<sub>1</sub>e<sup>jϕ<sub>1</sub></sup> + ... + w<sub>N-1</sub>e<sup>jϕ<sub>N-1</sub></sup></b>
</p>

<p align="justify">3. <b>A Âncora da Igualdade:</b> Ao definir que essa soma deve ser igual a <b>1</b>, forçamos o otimizador a garantir interferência construtiva máxima no alvo. Sem essa restrição, o algoritmo poderia simplesmente zerar todos os pesos para eliminar o ruído lateral, resultando numa antena sem potência de transmissão.</p>

<p align="justify"><h3>4. O Fluxo de Trabalho no Projeto</h3></p>

<p align="justify"><b>A. Definição da Geometria:</b> Define-se a posição física das antenas no espaço (linear, planar ou circular), criando a Matriz de Direção.</p>

<p align="justify"><b>B. Definição da Máscara:</b> O espaço é dividido em <i>Pass-band</i> (onde o sinal deve estar) e <i>Stop-band</i> (onde o sinal deve ser minimizado).</p>

<p align="justify"><b>C. Execução do Solver:</b> Traduzimos os requisitos matemáticos para o código:</p>

```python
# Exemplo de lógica CVXPY
objetivo = cp.Minimize(cp.norm(A_stopband @ w, "inf")) # Minimiza lóbulos laterais
restricao = [A_target @ w == 1] # Trava o ganho máximo no alvo

```

<p align="justify"><h3>5. Resultados</h3></p>

![Otimização de Antenas - Fig 1](https://raw.githubusercontent.com/rodfloripa/Otimizacao_de_Antenas/main/fig1.png)

![Otimização de Antenas - Fig 2](https://raw.githubusercontent.com/rodfloripa/Otimizacao_de_Antenas/main/fig2.png)

<p align="justify"><h3>Conclusão</h3></p>

<p align="justify">Projetar antenas com CVXPY transforma a "tentativa e erro" em uma <b>especificação de desempenho</b>. Você define como o lóbulo deve se comportar e o computador entrega o mapa exato (os pesos de amplitude e fase) para cada elemento da antena, garantindo que a física aconteça conforme planejado.</p>
