---
title: "Restore missing atoms in PDB file using Modeller"
date: 2022-10-31
categories:
  - Programming
  - Cheminformatics
tags:
  - Protein Structure Modelling
  - Modeller
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## What is comparative modelling
세상에 존재하는 아미노산은 다 합쳐 20개 뿐이 존재하지 않습니다.
각각의 아미노산이 어떤 화학적 구조를 가지고 있는지도 모두 알려져 있죠.
하지만 이러한 아미노산들이 수백개 연결되어 만들어지는 단백질의 구조는 쉽사리 예측하기 힘듭니다.
아미노산끼리의 연결부가 어떻게 접히는지에 따라 구조가 크게 달라질 수 있고, 단백질이 다른 물질과 상호작용하면서 얼마든지 구조가 변할 수 있기 때문입니다.
이러한 문제를 Protein Folding 문제라 합니다. 
2021년 7월 구글의 DeepMind가 인공신경망을 이용해 인간 단백질 2백만개의 구조를 모두 예측한 데이터의 이름이 AlphaFold인 이유도 Protein Folding이란 이름 때문입니다.

AlphaFold가 처음 나왔을 때는 이제 단백질과 관련한 대부분의 문제가 해결될 것이라 기대했습니다.
하지만 시간이 지남에 따라 AlphaFold의 정확도가 그다지 높지 않음이 밝혀지고 있습니다.
중심 구조는 잘 맞추지만, 2차 구조들의 세부적인 위치는 그다지 잘 맞추지 못하기 때문입니다. 
물론 2차 구조는 쉽게 변하기 때문에 '정답'을 맞추는 시험에서 채점기준으로 삼기는 까다로운 부분이 있었겠지만, 
단백질의 여러 생화학적 성질은 바로 이 2차 구조에 의해 결정되기 때문에 아쉬움이 남는 부분입니다. 

인공신경망을 이용해 복잡한 문제를 해결하는 것으로는 세계 제일이라 불리는 DeepMind도 제대로 해결하지 못하는 Protein Folding 문제.
그러면 기존의 학자들은 단백질 구조를 전혀 예측하지 못했을까요?
그것은 아닙니다. 
단백질 서열이 정해지면 구성되는 원자들과 그 결합이 정해지기 때문에 기본적으로 Molecular dynamics simulation을 이용해 에너지를 최소화하는 구조를 찾는 방식으로 예측할 수 있습니다.
물론 시스템에 존재하는 원소 개수가 수백개에서 수천개에 달하기 때문에 시간이 매우 많이 걸리고 복잡하다는 단점이 있죠.

그래서 고안된 간단한 방법이 바로 comparative modelling입니다. 
단백질의 전체 서열이 다르다해도 이를 구성하는 아미노산은 20개 뿐이 안되니, 서로 다른 단백질이더라도 부분적으로는 같은 서열을 공유하는 경우가 많습니다. 
그리고 이렇게 공유하는 부분서열의 실제 구조는 비슷할 수밖에 없는데요. 
따라서 미리 정해놓은 단백질 구조를 template으로 해서 원하는 아미노산 서열의 3차원 구조를 부분부분 예측하는 것이 가능합니다.
이러한 비교 예측법을 두고 comparative modelling이라 합니다. 

본 글에서는 comparative modelling 기능을 제공하는 대표적인 프로그램 [Modeller](https://salilab.org/modeller/)를 이용해 단백질 3차원 구조를 예측하는 [tutorial](https://salilab.org/modeller/tutorial/basic.html)을 따라가보고자 합니다. 
Tutorial에서는 TvLDH의 구조를 예측하기 위해 가능한 여러 구조를 검색하고, 그 중 몇 개를 동시에 시도해보면서 가장 높은 유사도를 보이는 구조를 고르는 방법을 설명하고 있습니다. 
저는 이와는 조금 다르게 단백질 `Cyclin-dependent kinase 2`(Uniprot id: [P24941](https://www.uniprot.org/uniprotkb/P24941/entry))의 APO 구조 [`5IF1`](https://www.rcsb.org/structure/5if1)를 template으로 삼아 단백질의 전체 구조를 복원하는데에 Modeller를 이용하고자 합니다. 
(APO 구조는 그 자체로 단백질의 구조정보를 촬영한 데이터이지만, 실험을 통해 측정한 구조다보니 빠져있는 원소나 아미노산이 많아 이러한 작업을 거치는 것입니다.)

## Basic Tutorial
### Prepare target sequence
먼저 원하는 아미노산 서열을 PIR 양식으로 만들어둘 필요가 있습니다.
`5IF1`은 chain이 여러개인 단백질인데, 그 중에서 A chain만 복원해봅시다. 
```
>P1;5IF1
sequence:5IF1:::::::0.00: 0.00
MENFQKVEKIGEGTYGVVYKARNKLTGEVVALKKIRLDTETEGVPSTAIREISLLKELNHPNIVKLL
DVIHTENKLYLVFEFLHQDLKKFMDASALTGIPLPLIKSYLFQLLQGLAFCHSHRVLHRDLKPQNLL
INTEGAIKLADFGLARAFGVPVRTYTHEVVTLWYRAPEILLGCKYYSTAVDIWSLGCIFAEMVTRRA
LFPGDSEIDQLFRIFRTLGTPDEVVWPGVTSMPDYKPSFPKWARQDFSKVVPPLDEDGRSLLSQMLH
YDPNKRISAKAALAHPFFQDVTKPVPHLRL*
```

줄을 하나하나 뜯어보자면 아래 줄은 해당 아미노산 서열의 아이디를 나타냅니다. 
```
>P1;5IF1
```

다음 줄은 아미노산 서열의 여러 메타정보를 담고 있는데, 여기서는 많이 생략된 것으로 이해하면 됩니다. 
```
sequence:5IF1:::::::0.00: 0.00
```
모든 정보가 다 담겨있는 예시는 아래와 같습니다. 
```
structureX:5fd1:1    :A:106  :A:ferredoxin:Azotobacter vinelandii: 1.90: 0.19
```
첫번째 칸은 해당 아미노산 서열의 측정 방법에 대한 정보이고, 이후로 각각 PDB id, 서열 중 사용하려는 부분의 시작과 끝의 번호와 chain,
단백질의 이름, 측정 해상도, R-factor 값을 의미합니다. 

마지막으로 아미노산 서열이 적혀있는데 이는 아래와 같은 규칙을 가지고 변환된 결과입니다. 
```
GLY: G  ALA: A  LEU: L  MET: M  PHE: F
TRP: W  LYS: K  GLN: Q  GLU: E  SER: S
PRO: P  VAL: V  ILE: I  CYS: C  TYR: Y
HIS: H  ARG: R  ASN: N  ASP: D  THR: T
```

### Alignment
Tutorial에서는 `Profile.build()`함수를 이용해 template으로 삼을만한 후보 단백질 구조를 찾고, 
`malign3d` 함수를 이용해 후보 중 최종 template을 결정하는 방법에 대해 설명했습니다.
하지만 이 글에서는 이미 있는 단백질 데이터를 이용해 더 정교한 단백질 구조를 복원해내는 용도로 사용하는 것이기 때문에,
이런 과정을 거치지 않고 주어진 단백질 데이터를 template으로 사용하면 됩니다. 
하지만 template으로 사용할 apo data가 residue가 비어있을 수도 있고, 번호가 이상하게 붙어있을 수 있기 때문에 곧바로 model building으로 넘어갈 수는 없습니다. 

model building에는 template과 target 모두의 서열을 포함하는 alignment 파일이 필요하고, 
이를 생성해주는 것이 바로 아래 코드입니다. 
``` python
from modeller import *

env = Environ()
aln = Alignment(env)

# 원하는 template 이름과 사용할 서열 정보
mdl = Model(env, file='5if1', model_segment=('FIRST:A','LAST:A'))   

# apo 구조의 pdb 파일
aln.append_model(mdl, align_codes='5if1A', atom_files='5if1.pdb')   

# Target sequence alignement file
aln.append(file='5if1.ali', align_codes='5if1')                     

# 가능한 gap의 최대 길이
aln.align2d(max_gap_length=50)

# Output
aln.write(file='5if1-ideal-apo.ali', alignment_format='PIR')
aln.write(file='5if1-ideal-apo.pap', alignment_format='PAP')
```
만들고자 하는 단백질의 pdb code(여기서는 `5if1`), pdb 파일 경로(`5if1.pdb`), target sequence ali 파일 (`5if1.ali`) 에 대해 바꿔 적습니다.
max_gap_length는 pdb 파일과 목표 서열을 align할 때, 서로 맞는 부분이 없어 건너뛸 수 있는 최대 길이를 말합니다. 
여기서는 tutorial대로 50으로 설정해두었습니다.

### Model building
이제 주어진 template을 이용해 목표 서열의 3D 구조를 만들어내는 과정만 남았습니다.
이는 아래와 같은 코드로 구현됩니다.
```python
from modeller import *
from modeller.automodel import *

env = Environ()

a = AutoModel(env, alnfile='5if1-ideal-apo.ali',
              knowns='5if1A', sequence='5if1',
              assess_methods=(assess.DOPE,
                              assess.GA341))
a.starting_model = 1
a.ending_model = 5
a.make()
```
`alnfile`에는 앞서 만들어둔 template과 목표 서열의 alignment 파일을 적어주면 되고,
`knowns`에는 template 이름, `sequence`는 목표 서열 이름을 넣으면 됩니다. 
`assess_methods`는 3D 구조 예측 결과를 평가하는 방법론을 입력하는 칸으로 이후 모델끼리 비교해 최종 결과물로 선택할 때 사용됩니다. 
이 부분은 tutorial에서 사용한 DOPE와 GA341을 그대로 옮겨왔습니다.

### Model evaluation
model을 총 5개 사용하기로 했기 때문에 결과도 5개 모델에 대해 나오게 됩니다. 
```
>> Summary of successfully produced models:
Filename                          molpdf     DOPE score    GA341 score
----------------------------------------------------------------------
5if1.B99990001.pdb            1650.57690   -36916.17969        1.00000
5if1.B99990002.pdb            1680.34778   -36939.61719        1.00000
5if1.B99990003.pdb            1609.59802   -36872.38672        1.00000
5if1.B99990004.pdb            1679.17847   -36643.48828        1.00000
5if1.B99990005.pdb            1546.36304   -37145.25781        1.00000
```
이제 이들 중 하나를 선택해 사용하면 되는데 여기에도 여러 방법이 있습니다.
단순히 DOPE 점수가 가장 낮은 model을 선택해도 되고, GA341 점수가 높은 경우를 선택해도 됩니다.
여기서는 GA341 점수는 모두 1로 나오므로, DOPE가 낮은 5번 모델을 선택할 수 있습니다.

## Comparison between before and after modelling
그럼 이러한 복원을 통해 `5if1`은 얼만큼 정보가 추가됐을지 확인해봅시다. 
물론 `5if1`은 애초부터 빠진 residue가 없는 데이터였기 때문에 많은 정보가 추가되지는 않았을텐데요. 
residue 내부에서 atom 단위로는 빠진 경우가 있어 이 부분을 검증해보았습니다.

Residue 39번 Threonine은 본래 pdb 상에서 N, C, CA, CB, CG2, O, OG1으로 구성되어 있어야하는 아미노산입니다. 
하지만 `5if1` pdb 파일에서는 N, CA, C, O, CB만 존재했죠.
이 부분이 잘 복원된 것을 확인할 수 있었습니다. 
```pdb
# 5if1.pdb
ATOM    312  N   THR A  39     -72.852 -28.579  12.797  1.00161.25           N  
ATOM    313  CA  THR A  39     -71.986 -29.787  12.911  1.00133.43           C  
ATOM    314  C   THR A  39     -71.912 -30.593  14.234  1.00134.00           C  
ATOM    315  O   THR A  39     -71.735 -31.807  14.102  1.00101.58           O  
ATOM    316  CB  THR A  39     -70.584 -29.588  12.336  1.00114.37           C  

# 5if1.B99990005.pdb
ATOM    304  N   THR A  39     -75.235 -28.513  13.811  1.00275.90           N
ATOM    305  CA  THR A  39     -74.457 -29.710  14.008  1.00275.90           C
ATOM    306  CB  THR A  39     -73.265 -29.832  13.091  1.00275.90           C
ATOM    307  OG1 THR A  39     -72.823 -31.182  13.052  1.00275.90           O
ATOM    308  CG2 THR A  39     -72.120 -28.947  13.587  1.00275.90           C
ATOM    309  C   THR A  39     -74.025 -30.133  15.389  1.00275.90           C
ATOM    310  O   THR A  39     -73.631 -31.294  15.497  1.00275.90           O
```