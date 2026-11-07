# Molde Base: Detector de Maos e Gestos com Python, MediaPipe e OpenCV

Este repositorio serve como um molde base de Visao Computacional para detetar maos em tempo real atraves da webcam. O projeto ja vem configurado com rastreamento avancado, reconhecimento de alguns sinais (como Libras e gestos divertidos) e um efeito visual interativo para interligar as maos.

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
import mediapipe as mp
from mediapipe.tasks import python
from mediapipe.tasks.python import vision

# Configuração do Hand Landmarker (Certifique-se de ter o hand_landmarker.task na pasta)
base_options = python.BaseOptions(model_asset_path='hand_landmarker.task')
options = vision.HandLandmarkerOptions(
    base_options=base_options,
    num_hands=2,  # Deteta até 2 mãos em simultâneo
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

    # Espelha a imagem horizontalmente (modo espelho)
    image = cv2.flip(image, 1)
    height, width, _ = image.shape

    # Converte para RGB e cria o objeto do MediaPipe
    image_rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    mp_image = mp.Image(image_format=mp.ImageFormat.SRGB, data=image_rgb)

    # Processa a detecção
    detection_result = detector.detect(mp_image)

    status_texto = "Mostre as maos..."

    # Se detetar 2 mãos, interliga os dedos correspondentes (Exemplo Base)
    if detection_result.hand_landmarks and len(detection_result.hand_landmarks) == 2:
        mao1 = detection_result.hand_landmarks[0]
        mao2 = detection_result.hand_landmarks[1]
        status_texto = "Duas maos interligadas!"

        pontos_para_ligar = [4, 8, 12, 16, 20] # Polegar ao Mindinho
        for idx in pontos_para_ligar:
            p1 = mao1[idx]
            p2 = mao2[idx]
            x1, y1 = int(p1.x * width), int(p1.y * height)
            x2, y2 = int(p2.x * width), int(p2.y * height)
            
            # Linha interligando as mãos
            cv2.line(image, (x1, y1), (x2, y2), (203, 192, 255), 2)
            cv2.circle(image, (x1, y1), 6, (203, 192, 255), -1)
            cv2.circle(image, (x2, y2), 6, (203, 192, 255), -1)

    # Se detetar 1 mão, deteta gestos de exemplo
    elif detection_result.hand_landmarks and len(detection_result.hand_landmarks) == 1:
        for hand_landmarks in detection_result.hand_landmarks:
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
                status_texto = "Mao detetada!"

            # Desenha os pontos da mão
            for landmark in hand_landmarks:
                x = int(landmark.x * width)
                y = int(landmark.y * height)
                cv2.circle(image, (x, y), 4, (203, 192, 255), -1)

    # Painel superior estilizado
    cv2.rectangle(image, (20, 20), (460, 90), (50, 20, 50), -1)
    cv2.rectangle(image, (20, 20), (460, 90), (203, 192, 255), 2)
    cv2.putText(image, status_texto, (35, 70), 
                cv2.FONT_HERSHEY_SIMPLEX, 0.7, (203, 192, 255), 2)

    # Mostra a janela
    cv2.imshow('Deteccao de Gestos', image)

    if cv2.waitKey(5) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
Use a sua criatividade
Este codigo e apenas um ponto de partida. Sinta-se totalmente livre para alterar as regras e criar novas funcionalidades, como:

Medir a distancia entre os dedos (criar uma regua digital).

Controlar o volume do computador ou o brilho da tela com os gestos.

Criar novos sinais da Lingua de Sinais (Libras) para traduzir frases.

Desenvolver mini-jogos controlados por movimentos das maos.

Como Executar
No seu terminal, com o ambiente virtual ativado, execute:

Bash
python hand_detection.py
Pressione a tecla q para fechar a camara a qualquer momento.

Desenvolvido por
Criado por Pamella Christiny (@codebypam).


---
3. Clica no botão verde **Commit changes...** no canto superior direito para salvar. 

O teu README vai ficar imediatamente limpo, bem estruturado e sem blocos amontoados!
