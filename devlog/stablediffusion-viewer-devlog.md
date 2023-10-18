# Stable Diffusion Image Viewer 에 대하여

[Stable diffusion image and meta info viewer](https://github.com/alongthecloud/sdimage_viewer)

## 개발환경 및 사용 라이브러리
Flutter 3.x 를 사용하였고 메타 데이터를 얻기 위해 외부 CLI 도구 exiftool를 사용하였다.

## 개발 후기
코드의 생산성은 좋았으나 UI 디자인 및 레이아웃에 고민을 더 많이 한 것 같고, 결국 가장 간단한 UI 구성으로 만들게 되었다.

시도하였으나 사용하지 않은 것
 - C++ 코드를 ffi 를 사용하여 연결하는 것, Image 패키지로 이미지 처리 및 텍스트 렌더링

## 핵심 코드
### 메타데이터 읽기
처음에는 .NET 의 메타데이터 라이브러리[Metadata Extrator .NET](https://github.com/drewnoakes/metadata-extractor-dotnet)을 사용하려고 하였으나 AOT-Native로 빌드했을 때 동작하지 않았다. 그래서, 외부 CLI 프로그램 exiftool을 사용하게 되었다.

exiftool은 json으로 결과 값을 출력하는 옵션이 있어서 프로세스를 호출한 다음 결과 값을 얻는 것이 어렵지 않았다.

```
    ProcessResult result = await Process.run(toolPath, [imagePath, '-j'],
        stdoutEncoding: utf8, stderrEncoding: utf8, runInShell: true);

      if (result.exitCode == 0) {
        return result.stdout.toString();
      } else {
        logger.info(result.stdout.toString());
        logger.log(Level.WARNING, result.stderr.toString());
      }
```
위 코드 처럼 호출을 하였고 exitCode 에 따라 성공과 실패를 판단하도록 했다.

### 이미지 불러오기

Flutter 에서 이미지를 불러오는 가장 간단한 방법은 Image.file 메서드를 쓰는 방법이다. 하지만, 메모리 캐싱을 달고 싶었고 그래서 메모리로 불러들인 다음 그것을 보여주는 방법을 찾게 되었다.

#### 처음 구현한 방법

메모리로 로드 후 그것을 Image.memory 를 사용해서 보여주는 방법

`Uint8List u8data = File(imagePath).readAsBytesSync()` 를 사용하여 메모리에 데이터를 불러들인 후 `Image.memory(u8data)` 를 사용하여 화면에 나타낸다. 이 방법은 화면에 보여줄 때 매번 디코딩을 하는 것으로 보이며, 다음 방법에 비해 덜 부드러운 느낌을 준다

#### 변경된 구현 방법

로드 후 디코딩하여 메모리에 저장 후 RawImage를 보여준다

```
    File file(imagePath);

    try {
      var byteData = await file.readAsBytes();
      ui.Codec result = await ui.instantiateImageCodec(byteData);
      ui.FrameInfo frame = await result.getNextFrame();
      image = frame.image;
    } catch (e) {
      return null;
    }

    return image;
```
현재 사용되고 있으며 이미지 파일을 로딩한 후 디코딩 된 Image 클래스를 얻고 그것을 `RawImage(image: image)` 를 사용하여 Flutter에서 보여준다.

### Markdown 라이브러리 사용

간단한 도움말 페이지를 작성하는데는 Markdown 을 사용하였고, [flutter_markdown](https://pub.devpackages/flutter_markdown) 라이브러리를 사용하였다.

플러터에는 내장되어 있는 풍부한 아이콘이 있는데 그것을 문서의 도움말에 넣고 싶었고, Markdown 위젯의 imageBuilder 인자를 사용하여 구현하였다.

마크다운 문서에는 `* ![help](icon://f0555) : Show this help.` 이렇게 서술하였다. (f0555는 Icons.question_mark 의 값)

해당 라이브러리에서 위 부분을 그릴 때 imageBuilder를 호출하게 된다.

```
  Widget _imageBuilder(Uri uri, String? title, String? alt) {
    if (uri.scheme == "icon") {
      int? iconID = int.tryParse(uri.authority, radix: 16);
      if (iconID != null)
        return Icon(IconData(iconID, fontFamily: 'MaterialIcons'));
    }

    return Text(alt ?? "undefined");
  }
```

이미지 빌더 함수에서 scheme값이 icon이면, 따라오는 16진수 코드 값을 파싱하여 해당 아이콘 위젯을 리턴한다. 그러면, 실행시 마크다운 문서에 아이콘 위젯이 표시되게 된다.

