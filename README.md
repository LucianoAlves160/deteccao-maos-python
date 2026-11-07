# Molde Base: Detector de Maos e Gestos com Python, MediaPipe e OpenCV

Este repositório contém uma aplicação prática de Visão Computacional para detecção e rastreamento de mãos em tempo real via webcam. O projeto conta com cálculo de distância dinâmica (Régua Digital entre polegar e indicador), reconhecimento de gestos em Libras e interligação visual entre articulações ao detectar duas mãos simultaneamente.

Sinta-se a vontade para usar esta estrutura como ponto de partida e explorar a sua criatividade para inventar novas funcoes, comandos ou jogos.

---

## O que voce precisa baixar / Instalar

Antes de comecar, certifique-se de ter instalado no seu computador:

1. **Python** (versao 3.10 ou superior recomendada).
2. O ficheiro de modelo oficial do MediaPipe necessario para a nova API de Tarefas:
   - Baixe o ficheiro **`hand_landmarker.task`** diretamente por este link oficial da Google: [Baixar hand_landmarker.task](https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task)
   - *Importante:* Coloque este ficheiro `.task` exatamente na mesma pasta onde estiver o seu script Python.

---

## Dependencias do Projeto

No seu terminal (PowerShell ou CMD), crie um ambiente virtual e instale as bibliotecas necessarias executando os seguintes comandos:

```bash
# Criar o ambiente virtual
python -m venv venv

# Ativar o ambiente virtual (no Windows PowerShell)
.\venv\Scripts\Activate

# Instalar as dependencias essenciais
pip install opencv-python mediapipe numpy
O Codigo Molde (hand_detection.py)
Crie um ficheiro chamado hand_detection.py na sua pasta e cole o codigo abaixo. Ele ja vem com a logica de interligar as maos e detetar alguns gestos como exemplo:

Python
import cv2
import math
import mediapipe as mp
from mediapipe.tasks import python
from mediapipe.tasks.python import vision

# Configuração do Hand Landmarker para até 2 mãos
base_options = python.BaseOptions(model_asset_path='hand_landmarker.task')
options = vision.HandLandmarkerOptions(
    base_options=base_options,
    num_hands=2,  # Permite detectar 1 ou 2 mãos
    min_hand_detection_confidence=0.5,
    min_hand_presence_confidence=0.5
)
detector = vision.HandLandmarker.create_from_options(options)

# Inicializa a captura de vídeo da webcam com DirectShow
cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)

print("Pressione 'q' na janela do vídeo para sair.")

while cap.isOpened():
    success, image = cap.read()
    if not success:
        print("Não foi possível ler o frame da câmera.")
        break

    # Espelha a imagem horizontalmente
    image = cv2.flip(image, 1)
    height, width, _ = image.shape

    # Converte para RGB e processa com o MediaPipe
    image_rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=image_rgb)
    detection_result = detector.detect(mp_image)

    status_texto = "Mostre as mãos..."

    if detection_result.hand_landmarks:
        qtd_maos = len(detection_result.hand_landmarks)

        # SE DETECTAR 2 MÃOS -> INTERLIGA OS DEDOS
        if qtd_maos == 2:
            mao1 = detection_result.hand_landmarks[0]
            mao2 = detection_result.hand_landmarks[1]
            status_texto = "Duas maos interligadas!"

            pontos_para_ligar = [4, 8, 12, 16, 20] # Polegar ao Mindinho
            for idx in pontos_para_ligar:
                p1 = mao1[idx]
                p2 = mao2[idx]
                x1, y1 = int(p1.x * width), int(p1.y * height)
                x2, y2 = int(p2.x * width), int(p2.y * height)
                
                # Linha e círculos interligando as duas mãos
                cv2.line(image, (x1, y1), (x2, y2), (203, 192, 255), 2)
                cv2.circle(image, (x1, y1), 6, (203, 192, 255), -1)
                cv2.circle(image, (x2, y2), 6, (203, 192, 255), -1)

        # SE DETECTAR 1 MÃO -> ATIVA A RÉGUA DIGITAL + GESTOS
        elif qtd_maos == 1:
            hand_landmarks = detection_result.hand_landmarks[0]

            # Ponto 4 (Polegar) e Ponto 8 (Indicador)
            polegar = hand_landmarks[4]
            indicador = hand_landmarks[8]

            x1, y1 = int(polegar.x * width), int(polegar.y * height)
            x2, y2 = int(indicador.x * width), int(indicador.y * height)

            # Cálculo da distância (Régua Digital)
            distancia_px = math.hypot(x2 - x1, y2 - y1)

            # Desenha a linha da régua digital entre polegar e indicador
            cv2.circle(image, (x1, y1), 7, (255, 0, 128), -1)
            cv2.circle(image, (x2, y2), 7, (255, 0, 128), -1)
            cv2.line(image, (x1, y1), (x2, y2), (255, 0, 128), 2)

            # Ponto médio para exibir a medida
            mid_x, mid_y = (x1 + x2) // 2, (y1 + y2) // 2
            cv2.putText(image, f"{int(distancia_px)} px", (mid_x - 20, mid_y - 10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2)

            # Reconhecimento de Gestos em Libras
            indicador_cima = hand_landmarks[8].y < hand_landmarks[6].y
            indicador_baixo = hand_landmarks[8].y > hand_landmarks[6].y
            medio_cima = hand_landmarks[12].y < hand_landmarks[10].y
            medio_baixo = hand_landmarks[12].y > hand_landmarks[10].y
            anelar_cima = hand_landmarks[16].y < hand_landmarks[14].y
            anelar_baixo = hand_landmarks[16].y > hand_landmarks[14].y
            mindinho_cima = hand_landmarks[20].y < hand_landmarks[18].y
            mindinho_baixo = hand_landmarks[20].y > hand_landmarks[18].y

            if mindinho_cima and indicador_baixo and medio_baixo and anelar_baixo:
                status_texto = "OI EM LIBRAS!"
            elif indicador_cima and medio_cima and anelar_baixo and mindinho_baixo:
                status_texto = "PAZ E AMOR!"
            elif indicador_cima and medio_cima and anelar_cima and mindinho_cima:
                status_texto = "TCHAU!"
            elif indicador_baixo and medio_baixo and anelar_baixo and mindinho_baixo:
                status_texto = "COMER!"
            else:
                status_texto = f"REGUA: {int(distancia_px)} px"

            # Desenha os pontos restantes da mão
            for landmark in hand_landmarks:
                x = int(landmark.x * width)
                y = int(landmark.y * height)
                cv2.circle(image, (x, y), 3, (203, 192, 255), -1)

    # Painel superior estilizado
    cv2.rectangle(image, (20, 20), (460, 80), (50, 20, 50), -1)
    cv2.rectangle(image, (20, 20), (460, 80), (203, 192, 255), 2)
    cv2.putText(image, status_texto, (35, 60), 
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (203, 192, 255), 2)

    # Exibe a janela de vídeo
    cv2.imshow('Deteccao de Maos e Gestos', image)

    if cv2.waitKey(5) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()


Como Executar
No seu terminal, com o ambiente virtual ativado, execute:

Bash
python hand_detection.py
Pressione a tecla q para fechar a camara a qualquer momento.

Desenvolvido por
Criado por Pamella Christiny (@codebypam).


## 🛠️ Funcionalidades Implementadas

* **Régua Digital:** Medição contínua da distância (em pixels) entre a ponta do polegar e a do indicador em tempo real.

---

Desenvolvido por **Luciano Alves** como projeto prático de Visão Computacional.
