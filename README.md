Este repositório contém um sistema RGB-D SLAM em tempo real, que é consciente de objetos, semântico e dinâmico, indo além dos sistemas tradicionais que geram apenas mapas baseados em geometria. Ele reconhece, segmenta e atribui rótulos de classe semântica a diferentes objetos na cena, enquanto os rastreia e reconstrói mesmo quando se movem de forma independente em relação à câmera.

Conforme uma câmera RGB-D escaneia uma cena complexa, a segmentação semântica em nível de instância baseada em imagem cria máscaras semânticas de objetos que permitem o reconhecimento em tempo real de objetos e a criação de uma representação em nível de objeto para o mapa do mundo. Ao contrário dos sistemas SLAM baseados em reconhecimento anteriores, Ele não requer conhecimento prévio ou modelos conhecidos dos objetos que pode reconhecer e pode lidar com múltiplos movimentos independentes. Em contraste com os sistemas SLAM habilitados para semântica mais recentes, que realizam a segmentação semântica em nível de voxel, Ele aproveita ao máximo a segmentação semântica em nível de instância para permitir que rótulos semânticos sejam fundidos em um mapa consciente de objetos. Apresentamos aplicações de realidade aumentada que demonstram as características exclusivas do mapa gerado por ele: consciente de instância, semântico e dinâmico.
[![Figure of MaskFusion](figures/teaser.jpg "Click me to see a video.")](http://visual.cs.ucl.ac.uk/pubs/maskfusion/MaskFusion.mp4)


## Construindo: 
O script `build.sh` mostra passo a passo como é construído e quais dependências são necessárias. As seguintes opções do CMake são obrigatórias: `PYTHON_VE_PATH`, `MASKFUSION_MASK_RCNN_DIR`, e é recomendado configurar `MASKFUSION_GPUS_MASKRCNN` também.

### CMake opções:
- `MASKFUSION_GPUS_MASKRCNN`: Lista de GPUs usadas pelo MaskRCNN, idealmente distintas da GPU usada pelo SLAM.
- `MASKFUSION_GPU_SLAM`: GPU utilizada pelo sistema SLAM, que deve ser a GPU usada pelo OpenGL.
- `MASKFUSION_MASK_RCNN_DIR`: Caminho para a sua instalação do [Matterport MaskRCNN](https://github.com/matterport/Mask_RCNN).
- `MASKFUSION_NUM_GSURFELS`: Número de surfels alocados para o modelo do ambiente.
- `MASKFUSION_NUM_OSURFELS`: Número de surfels alocados por modelo de objeto.
- `PYTHON_VE_PATH`: Caminho para o ambiente Python virtual (raiz), usado para o TensorFlow.

### Dependencias:
* Python3
* Tensorflow (>1.3.0, tested with 1.8.0)
* Keras (>2.1.2)
* MaskRCNN


## Rodando

- **Selecione as categorias de objetos** que você gostaria de rotular com o MaskRCNN. Para fazer isso, ajuste a matriz `FILTER_CLASSES` dentro de `Core/Segmentation/MaskRCNN/MaskRCNN.py.in`. Por exemplo, `FILTER_CLASSES = ['person', 'skateboard', 'teddy bear']` resultará no rastreamento de _skateboards_ e _teddy bears_. Na configuração atual, as regiões rotuladas como _person_ são ignoradas. Uma matriz vazia indica que todos os rótulos possíveis devem ser usados.

- O rastreamento de objetos individuais pode ser facilmente ativado/desativado chamando os métodos `makeStatic()` e `makeNonStatic()` das instâncias da classe `Model`. O sistema como um todo funciona de forma mais robusta quando os objetos são rastreados apenas quando estão sendo tocados por uma pessoa. Não estamos fornecendo software de detecção de mãos no momento.

## Conjunto de Dados e Ferramentas de Avaliação:

### Ferramentas
* Gravador para arquivos klg: [https://github.com/mp3guy/Logger2](https://github.com/mp3guy/Logger2)
* Visualizador para arquivos klg: [https://github.com/mp3guy/LogView](https://github.com/mp3guy/LogView)
* Conversor de imagens para klg: [https://github.com/martinruenz/dataset-tools/tree/master/convert_imagesToKlg](https://github.com/martinruenz/dataset-tools/tree/master/convert_imagesToKlg)
* klg para imagens/nuvens de pontos: [https://github.com/martinruenz/dataset-tools/tree/master/convert_klg](https://github.com/martinruenz/dataset-tools/tree/master/convert_klg)
* Avaliar segmentação (interseção sobre união): [https://github.com/martinruenz/dataset-tools/tree/master/evaluate_segmentation](https://github.com/martinruenz/dataset-tools/tree/master/evaluate_segmentation)
* Scripts para criar conjuntos de dados sintéticos com o Blender: [https://github.com/martinruenz/dataset-tools/tree/master/blender](https://github.com/martinruenz/dataset-tools/tree/master/blender)

## Hardware
Para executar sem problemas, você precisa de uma GPU rápida com memória suficiente para armazenar vários modelos simultaneamente. Utilizamos uma Nvidia TitanX na maioria dos experimentos, mas também testamos com sucesso o programa em um laptop com uma Nvidia GeForce™ GTX 960M. Se a memória da sua GPU for limitada, as opções CMake `MASKFUSION_NUM_GSURFELS` e `MASKFUSION_NUM_OSURFELS` podem ajudar a reduzir a pegada de memória por modelo (global/objeto, respectivamente).
Enquanto a etapa de rastreamento dele requer uma GPU rápida, o desempenho da segmentação baseada em movimento depende da CPU, e, portanto, ter um processador potente também é benéfico.


### Usando `cv::imshow` para debug

`cv::imshow(...)` requer a biblioteca `libopencv_highgui.so`, que pode (se GTK for usado) depender de `libmirprotobuf.so` e, portanto, de uma versão específica do *protobuf*. No entanto, o programa também vai exigir uma versão específica do *protobuf*, e pode acontecer que as duas versões entrem em conflito, levando a uma mensagem de erro como esta: *"Este programa requer a versão 3.5.0 da biblioteca de tempo de execução do Protocol Buffer, mas a versão instalada é 2.6.1. Atualize sua biblioteca. Se você compilou o programa sozinho, certifique-se de que seus cabeçalhos sejam da mesma versão do Protocol Buffers que sua biblioteca de tempo de link."*

A solução mais simples é compilar o OpenCV com `-DWITH_QT=ON`, o que remove a dependência do *protobuf* de `libopencv_highgui.so`.
### Crash (segfault) when loading python module ***MaskRCNN.py***
We noticed that loading the python module `MaskRCNN.py` can crash when the executable links to *hdf5* as this is potentially incompatible with the version required by *tensorflow*. Make sure to use the *opencv* library that is built in the deps subdirectory, which does not require linking to *hdf5*. (Set `OpenCV_DIR=<path>/deps/opencv/build` in cmake)

