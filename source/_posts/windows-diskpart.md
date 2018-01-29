---
title: DiskPart로 파티션을 내맘대로 다뤄보자.
date: 2018-01-30 01:24:59
tags:
    - windows
    - powershell
---

### 윈도우의 유일하게 유용한 시스템 유틸리티, Diskpart를 활용해보자. 

Windows의 CLI 지원은 한심한 수준이고 PowerShell은 있으나마나한 물건이지만 거의 유일하게 쓸만한 유틸리티인 Diskpart는 그 사용법을 확실히 알아두는 것이 좋다.

더구나 Windows 머신이라면 최소 일 년에 두어번 정도는 OS를 갈아엎고 디스크 환경을 바꿔줘야 할 일이 생기므로 Diskpart의 사용법을 잘 알아둔다면 Windows의 설치에 걸리는 시간과 노력(!)을 획기적으로 줄일 수 있을 것이다. 

#### list 명령 -> 뭔가 목록을 보여준다. 

말 그대로 list 하는 명령이다. 

```
list -[disk, partition, volume....]

list disk : 시스템에 설치된 물리적 디스크를 모두 보여준다. 
list partition: 선택된 디스크의 파팃녀을 모두 보여준다. 
list volume: 시스템에 등록되어 있는 모든 볼륨을 보여준다. 
```


#### select 명령 -> 뭔가를 선택한다. 

디스크의 구성 요소를 list 한 뒤엔 select를 수행해 수정할 파티션, 볼륨, 디스크 등을 확정해줘야 한다.

```
select disk 'disk_id' : disk_id번 디스크를 선택한다. 
select partition, volume, etc....
```

#### shrink -> 파티션의 크기를 줄인다.

여기부터 이제 실질적 파티션 정보의 조작을 전담하는 놈들을 알아볼 것이다. 

```
shrink 
    volume='drive_letter' : 드라이브 레터에 맞는 파티션의 크기를 줄일 것이다.
                            예를 들어 C드라이브의 크기를 줄이겠다면 shrink volume=c다.
                            그러나 이렇게 사용하는 것은 권하지 않는다.
                            가급적 list -> select -> shrink의 순서를 따르자. 
                            그래야 실수하지 않는다.
    
    querymax: 줄일 수 있는 최대 크기를 구하는 것이다. 
    예를 들어 500GB짜리 파티션이 230GB를 사용하고 있다면 querymax 값은 270GB다.

    desired='size_in_MB' : size_in_MB 크기만큼 볼륨을 줄인다. 
```

#### create -> 유에서 무를 창조한다.

파티션을 줄였다면 디스크에 노는 공간이 생겼을 터, CREATE을 돌려 파티션을 만들어줘야 파일을 저장할 수 있다. 

```
create
    - volume
    - partition
    - vdisk : 이건 아직 써보지 않아서 모르겠다. 위에 두 개는 각자 알아서 무슨 뜻일지 해석하자.
```

#### assign -> 이름표를 붙여준다. 

Windows는 모든 파티션에 드라이브 레터(e.g. C: D:)가 할당돼야 파일시스템 상에서 접근할 수 있다. 

assign 명령을 통해 당신의 꼴통같은 Windows 머신의 디스크에 최대한 굴욕적인 이름을 붙여주자. 

**assign 명령을 실행시키기 전에 DiskPart의 커서는 드라이브 레터가 할당되지 않은 빈 파티션을 향하고 있어야 한다. list - select 조합으로 잘 선택해주자.**

```
assign
    - 아무것도 안 쓰면 오름차순으로 자동 문자 배정. (e.g. C: D: 가 존재 -> E: 배정.)

    letter='your_desired_drive_letter' : 너의 마음대로 드라이브 레터를 붙여주는 것이다. 
    최대한 글자 모양이 우스꽝스러운 Z 나 Y 같은 녀석들을 추천한다. 
    Windows는 존중받을 가치가 없는 OS다.
```

#### 총정리 

총정리를 위해 아래와 같은 요구사항을 만족하는 사용 시나리오를 작성해 보겠다. 

*요구사항*
>1번 디스크의 2번 파티션의 크기를 500GB로 줄인다. 
>
>1번 디스크의 새로 생긴 공간에 파티션을 만들고 드라이브 레터를 자동 배정한다.
>
>NTFS 파일시스템으로 포맷하고 파일을 저장할 수 있게 준비한다.

```
list disk

select disk 1

list partition

select partition 2

shrink querymax

shrink desired=200000(파티션 2의 크기가 700GB였다고 가정하겠음. 1GB=1000MB)

create partition

assign

format fs=ntfs
```

**어때요, 참 쉽죠?**