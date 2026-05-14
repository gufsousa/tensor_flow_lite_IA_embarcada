# Relatório de Atividade Prática: IA Embarcada e Modelos Compactos

**Disciplina:** Inteligência Artificial Aplicada  
**Estudante:** Gustavo Fernandes  
**Projeto:** TensorFlow Lite Micro - Hello World no ESP32-S3

## 1. Reprodução dos passos do Hello World

O repositório implementa o exemplo `Hello World` do TensorFlow Lite Micro no ESP32-S3 usando ESP-IDF e simulação no Wokwi. O fluxo observado no projeto foi o seguinte:

1. Treinamento do modelo para aproximar a função `y = sen(x)` em um notebook.
   O diretório `notebooks/` contém os artefatos relacionados ao modelo.
2. Inclusão do runtime embarcado com a dependência `espressif/esp-tflite-micro`, declarada em [idf_component.yml](C:/Projetos/hello_world_tflite-main/hello_world/main/idf_component.yml).
3. Conversão do modelo `.tflite` para um array C/C++ compilável com `xxd`, gerando o arquivo [model.cc](C:/Projetos/hello_world_tflite-main/hello_world/main/model.cc).
4. Registro dos arquivos principais no build em [main/CMakeLists.txt](C:/Projetos/hello_world_tflite-main/hello_world/main/CMakeLists.txt), incluindo `main.cc`, `main_functions.cc`, `constants.cc`, `output_handler.cc` e `model.cc`.
5. Geração do firmware do ESP32-S3 com `idf.py build`.
6. Simulação no Wokwi com os arquivos [diagram.json](C:/Projetos/hello_world_tflite-main/hello_world/diagram.json) e [wokwi.toml](C:/Projetos/hello_world_tflite-main/hello_world/wokwi.toml), que apontam para a placa `ESP32-S3-DevKitC-1` e para os binários gerados em `build/`.

### Comandos de reprodução

```bash
idf.py add-dependency "espressif/esp-tflite-micro"
```

Conversão do modelo `.tflite` para `.cc` com `xxd` via WSL:

```bash
wsl xxd -i notebooks/hello_world_int8.tflite > main/model.cc
```

Build do projeto:

```bash
idf.py set-target esp32s3
idf.py build
```

Para executar no simulador Wokwi, o projeto usa:

- `diagram.json` para definir a placa e o monitor serial.
- `wokwi.toml` para mapear `build/hello_world.bin` e `build/hello_world.elf`.

## 2. Print da tela do Wokwi rodando o Hello World

A captura de tela da simulação foi adicionada abaixo a partir da pasta `assets`:

![Simulação do Hello World no Wokwi](assets/Captura_tela_simulacao.png)

## 3. Análise do código e da documentação

### Observações técnicas

- O arquivo `main/main_functions.cc` concentra a inicialização do modelo, a alocação da `tensor_arena`, a quantização da entrada e a dequantização da saída.
- A inferência usa `tflite::MicroInterpreter`, abordagem adequada para microcontroladores por evitar dependências pesadas e trabalhar com memória estática.
- O projeto registra somente a operação `FullyConnected` em `MicroMutableOpResolver<1>`, o que reduz o tamanho do binário final e é uma boa prática em IA embarcada.
- A `tensor_arena` foi definida com `2000` bytes. Esse valor funciona para o exemplo, mas é um ponto sensível em sistemas embarcados: qualquer troca de modelo ou operador pode exigir reavaliação de memória.
- A entrada e a saída usam quantização `int8`, reduzindo uso de memória e custo computacional em relação a `float32`, com pequeno impacto de precisão quando comparado ao benefício em dispositivos restritos.
- A função `HandleOutput` em `main/output_handler.cc` envia os pares `x` e `y` para o monitor serial com `MicroPrintf`, o que simplifica a validação no Wokwi.
- O `diagram.json` está enxuto e suficiente para a simulação, conectando apenas a UART ao monitor serial. Isso é coerente com o objetivo didático do exemplo.

### Pontos de atenção

- O `README` anterior estava com problema de codificação de caracteres. Isso foi corrigido nesta versão para facilitar leitura e manutenção.
- O repositório foi inicializado com git nesta pasta, então os arquivos já estão prontos para versionamento local.
- Não encontrei, neste turno, um log de execução salvo em arquivo dentro do repositório. A validação visual fica representada pela captura da simulação em `assets/Captura_tela_simulacao.png`.

## Conclusão

O projeto demonstra corretamente um fluxo clássico de IA embarcada: modelo pequeno, inferência local, uso de memória estática e integração com um ambiente de simulação para validar o comportamento sem hardware físico. Como base acadêmica e didática, a estrutura está adequada e evidencia bem as restrições e decisões típicas de TinyML no ESP32-S3.
