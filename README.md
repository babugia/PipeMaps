

# Mapa de encanamentos feito com Realidade Aumentada
>  Por Guilherme Babugia. 08 de Agosto de 2018

## Introdução

Neste artigo vamos ver como Realidade Aumentada funciona e mostrar como foi aplicada em um projeto de uma casa de verdade, utilizando o ARKit da Apple. Com o aplicativo Pipe Maps aberto, podemos andar pela casa e apontar a câmera para onde desejamos, visualizando assim, através do Iphone, os encanamentos e conduites instalados na casa.

## O que é Mapa de Encanamentos?

Também chamado de planta hidráulica, o mapa de encanamentos se encontra na planta do imóvel e ele mostra o conjunto de ramificações do encanamento e os lugares por onde passam estes canos, com tamanhos e outros detalhes. Neste mapa de encanamentos também foi considerado conduites responsaveis pela distribuição de eletricidade.

## O que é Realidade Aumentada?
A realidade aumentada é um elemento das novas tecnologias que dispõe de uma visão diferente da realidade, usa seu ambiente natural existente e simplesmente sobrepõe informações virtuais sobre ele a partir da câmera de um dispositivo, de modo que esses elementos parecem habitar a realidade, nada mais é que a ilusão que o conteúdo virtual faz parte do mundo real.

## O que é ARKit?
É um framework da Apple usado para construir aplicações que utilizem AR (Augmented Reality), o seu uso faz com que o desenvolvedor não tenha necessidade de mexer com marcadores de localização, inicializar câmeras, tratar cálculos com profundidade, sem necessidade de configuração externa, etc.
O ARKit tem a capacidade de criar e rastrear uma correspondência entre o espaço do mundo real que o usuário habita e um espaço virtual onde pode-se modelar o conteúdo visual.
Para criar tal correspondência, o ARKit usa uma tecnologia chamada visual-inertial odometry (VIO) para rastrear com precisão e em tempo real a posição no mundo ao redor, aproveitando dos dados do sensor da câmera com informações do CoreMotion. Permitindo assim que o dispositivo perceba como ele se move com alta precisão e sem necessidade de calibração.
Visual-inertial odometry é uma tecnologia que utiliza as imagens da câmera e core motion para conseguir com precisão onde o celular está localizado e como está orientado.
O ARKit analisa o que a câmera mostra, detecta planos horizontais e também estima a quantidade total de luz disponível e aplica a quantidade correta de iluminação nos objetos virtuais.
Para garantir que os objetos virtuais continuem com a aparência que estão no mundo real, a posição do celular é recalculada entre cada frame que é recarregado e mostrado na tela do dispositivo, isso ocorre cerca de trinta ou mais vezes por segundo. Estes cálculos são feitos duas vezes, em paralelo. A posição é rastreada através da câmera, combinando um ponto no mundo real com um pixel no sensor da câmera de cada quadro. Além disso, a posição também é rastreada pelo sistema inertial (acelerômetro e giroscópio usados em conjunto, os quais são denominados Unidade de Medição Inercial ou IMU). A saída de ambos os sistemas é então combinada através de um Filtro Kalman, método matemático que determina ao longo do tempo com diversas amostras contaminadas com ruído e outras incertezas quais dos dois sistemas fornecem a melhor estimativa de sua posição “real” (referido como Ground Truth). Após tudo isso, a posição atualizada é mostrada através do SDK ARKit. Assim como o odômetro do carro determina a distância do carro percorreu, o sistema VIO rastreia a distância que seu iPhone percorreu no espaço 6D. 6D significa 3D de movimento xyz (translação), além de 3D de roll / pitch / yaw (rotação).

## Montando a casa
Para montar a casa, foi utilizado o Unity 3D, que se apresenta como um Game Engine (motor de jogo). Foi utilizado vários assets (representação de qualquer item que possa ser usado no projeto), por exemplo, porta, sofa, geladeira, etc. Assim foi sendo construida a casa virtual, baseando-se na casa real. Uma das dificuldades do projeto, foi a “real” posição e tamanho dos objetos encontrados dentro da casa, desde objetos grandes como geladeira, sofa, até objetos pequenos, como talheres e interruptores, a construção virtual da casa tem que ser bem parecido com a realidade, pois quando o usuário abre o aplicativo, tem que sentir realmente que está dentro da casa, tanto no espaço real, quando no espaço virtual. Como o ARKit faz o trabalho pesado, tirando o trabalho do desenvolvedor lidar com câmeras, calculos e qualquer configuração sobre AR, o maior trabalho, além de ajustar a casa e seus objetos (tamanho e posição), foi fazer com que os encanamentos e conduites fossem visto como uma visão de “raio-x” e paredes e outros assets, como pias, tivessem a capacidade de deixar enxergar, por trás deles, os canos e conduites em modo raio-x.

## Criando o modo “raio-x”
Para deixar qualquer asset ter a capacidade de deixar ver possiveis objetos por trás deles, foi criado um shader (Shaders são pequenos scripts que contêm os cálculos matemáticos e algoritmos para calcular a cor de cada pixel renderizado, com base na entrada de iluminação e na configuração do Material) específico chamado `Diffuse-Stencil-Write.shader`.

```swift
Tags { "RenderType" = "Opaque" }
LOD 200
    
Stencil {
    Ref 1
    Comp Always
    Pass Replace
    ZFail Keep
} 

```
Este trecho de código é o mais crucial do script, basicamente estou dizendo que este shader poderá ser substituido, o LOD 200 indica que o shader se propaga em todas direções do objeto, LOD (Level Of Detail) significa nível de detalhe. Stencil é um buffer que é usado como finalidade geral para salvar ou descartar pixels, dentro dele estou usando o valor '1' para comparação, dizendo que sempre vou comparar, e que se o teste de stencil passar, eu vou substituir, por ultimo, estou dizendo que se mesmo que o teste de stencil passar e o teste de profundidade falhar, eu vou manter o valor.

Para deixar qualquer asset ter a aparencia de raio-x, foi criado um shader específico chamado `ColoredOutline.shader`.

```swift
Stencil {
    Ref 0
    Comp NotEqual
}

Tags {
    "Queue" = "Transparent"
    "RenderType" = "Transparent"
    "XRay" = "ColoredOutline"
}

ZWrite Off
ZTest Always
Blend One One 
```


Este trecho de código é o mais crucial do script, o stencil está usando o valor '0' para comparação e o comp como NotEqual, indica que só irá renderizar os pixels, cujo o valor de referencia seja diferente do valor que está no buffer. Usando a tag queue eu estou dizendo que qualquer shader Transparente garante que eles sejam desenhados após todos os objetos opacos e assim por diante, o = transparent significa que essa fila de renderização é renderizada após qualquer objeto geométrico. O renderType = transparent fala que o que será renderizado, será semitransparente. O ZWrite controla se os pixels deste objeto são gravados no buffer de profundidade. Para objetos sólidos o correto é deixar ativado (On), se estiver desenhando efeitos semitransparentes o correto é deixar desativado (Off). O ZTest Always indica que o teste de profundidade sempre deve ser feito. Blending é usado para fazer objetos transparentes, o comando blend controla como eles são combinados com o que já está la. 
 
#### *Resumindo..*

O Diffuse-Stencil-Write.shader, que é o shader dos objetos que poderei enxergar os encanamentos e conduites por trás deles, fala que o shader pode ser substituido e o seu “valor” é 1, e o ColoredOutline.shader que é o shander dos encanamentos e conduites para ficarem com efeito raio-x, fala que vou renderizar com efeito semitransparente e que será renderizado após todos os outros objetos “normais” serem renderizados e seu valor é 0, então quando a câmera apontar pra uma parede por exemplo, seu valor de shader é 1, se tiver um objeto com efeito raio-x atrás dela, dará para enxergar pois seu valor de shader é 0, então substituirei o shader padrão da parede, pelo do objeto (encanamento ou conduite), ficando visivel através do aplicativo.

## Demonstração

Neste video temos uma demo do aplicativo usado na cozinha da casa.
<p>
    <img src=./media/kitchen2.gif width="600px" height="338px">
</p>

Neste video temos uma demo do aplicativo usado no banheiro da casa.
<p>
    <img src=./media/bathroom_together.gif width="600px" height="338px">
    
</p>

Neste video temos uma demo do aplicativo usado no quarto da casa.
<p>
    <img src=./media/bedroom.gif width="600px" height="338px">
</p>

## Conclusão

Com o uso do AR, deu para se ter uma ideia das infinitas possibilidades de aplicações que podem usar essa tecnologia, creio que Realidade aumentada está entre as tecnologias que vão impactar o mundo em um futuro próximo, tecnologias estas como, Internet das coisas, Realidade virtual, Big Data, Carros autônomos entre outras.
Como é uma tecnologia praticamente recente, ela possui algumas limitações, como por exemplo, se movimentar rapidamente o Iphone (não precisa ser muito rapido), a aplicação "se perde" e as posições dos objetos ficam bagunçadas, o mesmo se passar alguém na frente da câmera. 
Uma idéia relativamente simples, como poder saber onde estão instalados o encanamento e conduites da sua casa, sem precisar procurar a planta do imóvel, ter o trabalho de achar, no meio de tantas informações, onde passa algum cano na sua parede, pra poder pregar algum prego para pendurar algo, simplesmente abrindo um aplicativo no seu celular. É um exemplo de aplicação útil e que no final das contas, fará com que o usuario não perca tanto tempo procurando um documento, que provavelmente não lembra onde está guardado.
