# Test_StencilShader
Test_StencilShader For Silhouette

<img width="1440" alt="StencilShaderResult" src="https://user-images.githubusercontent.com/58582985/173217221-7331a9bc-a52f-4f14-9865-ad265fea3348.png">

![image](https://user-images.githubusercontent.com/58582985/173260983-df5d5371-cdfe-4cdc-bcda-c21d25511042.png)

![image](https://user-images.githubusercontent.com/58582985/173261794-80684739-c259-4e87-833c-efa820a678f1.png)

![image](https://user-images.githubusercontent.com/58582985/173262377-7626fde8-a2a9-4cc6-a6ae-634b93fc3b5b.png)

----------------------
# MP 프로젝트에 적용결과

### Ref Value : Player = 3, Obstruction = 2  

#### Step 1
Player(Ref 3)가 그려진 이후(sorting에 의해서 그려지는 순서가 결정된 것)에 그려지는 Obstruction(Ref 2)은 일단 항상 그려져야하므로 스텐실 비교함수가 Always이어야한다  
그렇다보니 Always에 의해 플레이어의 Ref값이 바로 덮어지게 된다 그렇기에 플레이어의 Ref는 어디에서도 찾아볼 수 없다  
```ShaderLab
Stencil
{
  Ref 2
  Comp Always
  Pass Replace
}
```
![img1](https://user-images.githubusercontent.com/58582985/209546864-136efb07-4e3c-472f-8abb-999b92cc6391.png)

#### Step 2
장치가 그려지면서 플레이어가 있는 부분은 ref를 2로 RePlace하고, 그렇지 않는 부분은 3 그대로 둔다 (이후에 장치의 스텐셀 ref값도 쓰일 수 있기때문)

그런데 이렇게 할 경우 문제가 생긴다. Comp 함수가 Less이므로 Player가 있는 곳에만 Obstruction이 그려지는 것이다
```ShaderLab
Stencil
{
　　Ref 2
　　Comp Less
　　Pass Replace
}
```
![img2](https://user-images.githubusercontent.com/58582985/209546867-c033ad73-0044-4be6-a5ee-78cb65d924fb.png)


그래서 나는 다음과 같은 상황을 원했다 일단 그림은 다그려 (Comp Always)
그리고 플레이어보다 Ref가 작으면 (Comp Less) 그곳만 스텐실 버퍼를 변경해(Pass Replace)

결국 두가지의 Comp를 사용해야한다는 의미.. 이것이 가능한가? 사실 나는 셰이더를 겉핥기식으로 공부했기에 모른다
멀티 패스든.. 뭐든(멀티패스는 URP에서 지원이 되지 않는다고 하는 것같다, 그릭:메모리즈 오브 아주르 또한 빌트인 렌더러 파이프라인이다)

해결책은 거진 3주를 고민한것같다 이게 되다니...
그냥 스프라이트를 두개 두는 것이다 첫번째는 그냥 그리는 No Stencil 스프라이트
두번째는 스텐실이 적용된 스프라이트이다 이미 첫번째 스프라이트에서 그림을 다 그렸으니 플레이어와 겹쳐진 부분만 그려져도 무방하다 그리고 그곳은 스텐실 버퍼가 2이다 그렇게 필터에 의해 스텐실 버퍼가 2인 곳이 검출되는 것이다 (그림 3)

------------
추가로 블루프린트를 적용했을때 장치의 Ref값도 활용해야한다면 두번째 장치 스프라이트의 그려지지 않은 부분의 스텐실 버퍼값을 이용하면 된다

Stencil
{
　　Ref 2
　　Comp Less
　　Pass Replace
　　Fail IncrSat 　　　　//플레이어가 없는 부분은 Ref가 0 이므로 IncrSat을 이용하여 스텐실 버퍼값을 1로 설정한다
}

그리고 Ref 1을 검출하는 필터, 2를 검출하는 필터 두개를 생성하면 되겠다(그림4)

![img3](https://user-images.githubusercontent.com/58582985/209546874-3f48a838-aa0b-4ca7-bfc9-14a46fac906b.png)
![img4](https://user-images.githubusercontent.com/58582985/209546878-d900f1e5-4747-434f-80a9-19cd9f6a90ee.png)
