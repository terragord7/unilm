# [DeltaLM](https://arxiv.org/abs/2106.13736)

**Encoder-Decoder Pre-training for Language Generation and Translation** 

[DeltaLM: Encoder-Decoder Pre-training for Language Generation and Translation by Augmenting Pretrained Multilingual Encoders.](https://arxiv.org/abs/2106.13736) Shuming Ma, Li Dong, Shaohan Huang, Dongdong Zhang, Alexandre Muzio, Saksham Singhal, Hany Hassan Awadalla, Xia Song, Furu Wei. CoRR abs/2106.13736.

- September 2021: DeltaLM ranks first on the [WMT21 multilingual translation task](http://www.statmt.org/wmt21/large-scale-multilingual-translation-task.html).
- August 2021: release code and pretrained checkpoints.

---

## Pretrained Models

- [DeltaLM-base](https://deltalm.blob.core.windows.net/deltalm/deltalm-base.pt): #enc-dec=12-6; #hidden=768; #head=12; #FFN=3072 (#parameters: 360M)
- [DeltaLM-large](https://deltalm.blob.core.windows.net/deltalm/deltalm-large.pt): #enc-dec=24-12; #hidden=1024; #head=16; #FFN=4096 (#parameters: 830M)
- [Vocabulary](https://deltalm.blob.core.windows.net/deltalm/dict.txt) and [Sentencepiece-model](https://deltalm.blob.core.windows.net/deltalm/spm.model)
- DeltaLM can be finetuned to support language generation and translation tasks for **100+ languages**


## Cross-lingual Abstractive Summarization - [Wikilingua](https://arxiv.org/abs/2010.03093)

We evaluate DeltaLM on cross-lingual abstractive summarization benchmark. We report the results by averaging the numbers in different languages. 

|   Model   |   #Params   |  ROUGE-1  |  ROUGE-2  |  ROUGE-L  |
|-----------|-------------|-----------|-----------|-----------|
| [mBART](https://arxiv.org/abs/2001.08210)     | 610M        | 34.5      | 12.9      | **28.7**      |
| [mT5](https://arxiv.org/abs/2010.11934)       | 300M        | 27.5      | 8.8       | 22.8      |
| [mT5](https://arxiv.org/abs/2010.11934)       | 580M        | 31.8      | 11.5      | 26.0      |
| DeltaLM   | 360M        | **35.3**      | **13.4**      | **28.7**      |


## Setup

```bash
cd src/
pip install --editiable ./
```

## Fine-tuning

1. Organize the raw data in the following structure:
```
.
+-- /path/to/data/
|   +-- train.src
|   +-- train.tgt
|   +-- valid.src
|   +-- valid.tgt
```

2. Tokenize the data using [Sentencepiece](https://github.com/google/sentencepiece):

```bash
spm_encode --model=/path/to/checkpoint/spm.model --output_format=piece < train.src > train.spm.src
spm_encode --model=/path/to/checkpoint/spm.model --output_format=piece < train.tgt > train.spm.tgt
spm_encode --model=/path/to/checkpoint/spm.model --output_format=piece < valid.src > valid.spm.src
spm_encode --model=/path/to/checkpoint/spm.model --output_format=piece < valid.tgt > valid.spm.tgt
```

3. Binary the data:

```bash
data_bin=/path/to/data-bin/
python ./fairseq_cli/preprocess.py  \
    --trainpref train.spm \
    --validpref valid.spm \
    --source-lang src --target-lang tgt \
    --destdir $data_bin \
    --srcdict /path/to/checkpoint/dict.txt \
    --tgtdict /path/to/checkpoint/dict.txt \
    --workers 40
```

4. Fine-tuning:

```bash
PRETRAINED_MODEL=/path/to/checkpoint/model.pt
LANGS="src,tgt"
LANG_PAIRS="src-tgt"
python train.py $data_bin \
    --save-dir checkpoints/ --arch xlmt_decoder_variant_large_from_deltalm_postnorm --pretrained-deltalm-checkpoint $PRETRAINED_MODEL --init-encoder-only --init-decoder-only --variant addffn \
    --task translation_multi_simple_epoch --sampling-method "linear" --sampling-temperature 5.0 --min-sampling-temperature 1.0 --warmup-epoch 5 \
    --encoder-langtok "tgt" --langtoks '{"main":("tgt",None)}' --langs $LANGS --lang-pairs $LANG_PAIRS \
    --share-all-embeddings --max-source-positions 512 --max-target-positions 512 --criterion label_smoothed_cross_entropy --label-smoothing 0.1 \
    --optimizer adam --adam-betas '(0.9, 0.98)' --lr-scheduler inverse_sqrt --lr 1e-4 --warmup-init-lr 1e-07 --stop-min-lr 1e-09 --warmup-updates 4000 \
    --max-update 400000 --max-epoch 100 --max-tokens 4096 --update-freq 1 \
    --seed 1 --log-format simple --skip-invalid-size-inputs-valid-test --ddp-backend=no_c10d
```
**Note: Please adjust the `max-tokens` and `update-freq` to suit in different experimental environments. Recommendation of the total batch size is `4096 * 128` tokens per step.

---

## Citation

If you find this repository useful, please consider citing our work:
```
@article{deltalm,
      title={{DeltaLM}: Encoder-Decoder Pre-training for Language Generation and Translation by Augmenting Pretrained Multilingual Encoders}, 
      author={Shuming Ma and Li Dong and Shaohan Huang and Dongdong Zhang and Alexandre Muzio and Saksham Singhal and Hany Hassan Awadalla and Xia Song and Furu Wei},
      year={2021},
      eprint={2106.13736},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```

## Acknowledgement

This repository is built using the [Fairseq](https://github.com/pytorch/fairseq) repository.

## License
This project is licensed under the license found in the LICENSE file in the root directory of this source tree.

[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct)

### Contact Information

For help or issues using DeltaLM models, please submit a GitHub issue.

For other communications related to DeltaLM, please contact Shuming Ma (`shumma@microsoft.com`), [Furu Wei](http://gitnlp.org/) (`fuwei@microsoft.com`).
