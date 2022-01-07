<div align="center">
  
# Foreigner's Korean speech recognition of voice
  
### Korean STT 모델 개발

___
</div>

한국어 음성인식 오픈소스 툴킷인 [kospeech](https://github.com/sooftware/kospeech)를 활용하여 한국어 음성 인식 모델을 개발한 내용을 다루었습니다. 
자세한 설명은 [블로그](https://velog.io/@letgodchan0/%EC%9D%8C%EC%84%B1%EC%9D%B8%EC%8B%9D-%ED%95%9C%EA%B5%AD%EC%96%B4-STT-1)를 참고해 주시면 감사하겠습니다.

</br></br>
## Summary
- NIA와 CSLEE 주관하에 외국인이 발화하는 한국어 음성 데이터 30만개를 활용하여 음성 인식 모델을 개발하고 이를 통해 아이디어를 제안하는 프로젝트 내용입니다.
- kospeech를 선정한 이유는 외국인 발화 같은 경우 내국인 발화 음성과는 구분되는 특성이 있을 것이라는 여러 연구 결과에 근거하여 외국인 발화 음성만으로 학습시킨 모델을 만들고 그 성능을 Pre-trained 모델과 비교해 볼 필요가 있었습니다. 
- 또한 실제 한국어 음성으로 구현된 STT 모델이 있는지 여부를 고려했는데, 대부분의 오픈소스 STT 모델들은 그 성능이 영어에 한정해서 알려져 있었기 때문에 한국어 음성에 대해서도 검증된 사례가 있는지 여부가 중요했습니다.
- 그 결과 kospeech가 제공하는 다양한 어쿠스틱 모델 중에서도 성능이 가장 우수한 것으로 알려진 deepspeech2를 베이스 모델로 선택했고 전처리 방식은 character unit으로 진행했습니다. 
- kospeech를 활용하는 과정에서 발생한 에러들을 해결하거나 혹은 제 데이터와 환경에 맞도록 최적화 시켰기 때문에 원본 오픈소스와 다른 내용이 많습니다.
- 저희 팀은 1만 개의 테스트 데이터에 대해 CER, WER을 측정한 결과 'CER: 0.085, WER: 0.1919'의 결과를 얻었고 전체 참가 팀 중 성능 부문 1위, 최종 평가 2위로 우수상을 수상했습니다.
- ./etc 파일의 경우 대회 포스터 및 발표 자료를 업로드 했고 ./KoreanSTT 폴더에 전반적인 내용을 업로드 했습니다. 
</br></br>

## Module Installation
- 전처리, 학습, 예측, 예측한 결과 저장에 필요한 모든 모듈을 포함 시킨 파일로 가상환경을 생성하고 Python 3.8을 사용했습니다.
- 저희 팀원의 경우 cuda 11.2 버전이 제공하는 pytorch 버전과 호환이 되지 않아 pytorch만 재설치를 진행했습니다.
```
pip install -r requirements.txt
```
</br></br>

## Preprocessing
- output_unit은 kospeech가 제공하는 3가지 방식을, preprocess_mode는 '%'를 읽는 방식('퍼센트' 또는 '프로')을 선택하는 것으로 상황에 맞게 지정하시면 됩니다.
- preprocess.py 파일을 실행하기 위해 '오디오 파일 경로' + '\t' + '한국어 전사'의 형식으로 텍스트 파일을 만들어 주셔야 합니다.
- 전처리의 경우 unit별로 사전을 생성하게 되고 이를 통해 학습에 필요한 transcript.txt 파일을 생성하게 됩니다.
```
!python ./dataset/kspon/main.py --dataset_path $dataset_path --vocab_dest $vacab_dict_destination --output_unit 'character' --preprocess_mode 'phonetic' 
```
</br></br>

## Train
- 학습에 실행코드를 실행하기 위해서는 옵션(epoch, batch_size, spec_augment emd)을 수정하거나 이외에도 변경해주어야 할 사항들이 있습니다.
- 자세한 사항은 [블로그](https://velog.io/@letgodchan0/%EC%9D%8C%EC%84%B1%EC%9D%B8%EC%8B%9D-%ED%95%9C%EA%B5%AD%EC%96%B4-STT-3)를 참고해주시면 감사하겠습니다.
```
!python ./bin/main.py model=ds2 train=ds2_train train.dataset_path=$dataset_path
```

</br></br>

## Inference

모든 command에 대한 deivice 옵션은 상황에 맞게 지정해주세요.

1. 음성 파일 1개에 대한 예측
* Command

```
!python ./bin/inference.py --model_path $model_path --audio_path $audio_path --device "cpu"
```
* Output

```
음성 인식 결과
```
2. 음성 파일 1개에 대한 예측과 Cer, Wer 계산 결과 저장</br>
(결과는 dst_path에 저장되며, 정답 label인 transcripts.txt파일을 transcript_path에 지정해주어야 합니다. 그 형식은 전처리에 필요한 train.txt 파일 혹은 학습에 사용되는 transcripts.txt와 동일해야 합니다.)
* Command
```
python ./bin/inference_wer.py --model_path $model_path --audio_path $audio_path --transcript_path $transcript_path --dst_path $result_destination --device "cpu"
```
* Output

```
음성 인식 결과
```
3. 음성 파일 여러 개(폴더)에 대한 예측과 그 결과 저장(.txt, .xlsx)
* Command
```
python ./bin/prediction.py --model_path $model_path --audio_path $audio_path --submission 'True' --device "cpu"
```
'submission = True'로 지정하면 예측 결과를 .xlsx 파일로 저장할 수 있습니다. 다만 2개의 컬럼을 갖는 제출용 excel 파일을 필요로 합니다.
* Output

./outputs 폴더에 .txt와 .xlsx 파일 생성
</br></br>

## - References
- kospeech:
https://github.com/sooftware/kospeech
