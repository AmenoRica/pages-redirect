# Pages.dev Redirect

`?red=asdf`를 `https://asdf.pages.dev/`로 연결하는 정적 GitHub Pages 리다이렉터입니다.

예시: <https://r.ameno.cc/?red=asdf>

## 압축된 URL로 이동하기

`?enc=<encoded>` 형식으로 HTTP 또는 HTTPS URL을 전달할 수도 있습니다. `enc`와 `red`가 함께 있으면 `enc`가 우선합니다.

파라미터 없이 페이지에 접속하면 URL 입력란에서 `enc` 링크를 바로 만들 수 있습니다.

`encoded` 값은 다음 순서로 만듭니다.

1. 원본 URL을 UTF-8 바이트로 변환합니다.
2. 바이트를 gzip으로 압축합니다.
3. 압축 결과를 Base64로 인코딩합니다.
4. Base64의 `+`를 `-`, `/`를 `_`로 바꾸고 끝의 `=` 패딩을 제거합니다.

브라우저에서는 이 과정을 반대로 수행한 다음 복원된 URL로 이동합니다. 외부 라이브러리는 사용하지 않으며, 브라우저의 `CompressionStream`과 `DecompressionStream`을 사용합니다.

### `enc` 값 만들기

아래 함수를 최신 브라우저의 개발자 도구 콘솔에서 실행한 뒤 `encodeUrl('https://example.com/path?hello=world')`처럼 호출합니다.

```js
async function encodeUrl(url) {
  const target = new URL(url);
  if (target.protocol !== 'http:' && target.protocol !== 'https:') {
    throw new Error('HTTP 또는 HTTPS 주소만 사용할 수 있습니다.');
  }

  const stream = new Blob([new TextEncoder().encode(target.href)])
    .stream()
    .pipeThrough(new CompressionStream('gzip'));
  const bytes = new Uint8Array(await new Response(stream).arrayBuffer());
  let binary = '';
  for (const byte of bytes) binary += String.fromCharCode(byte);

  return btoa(binary)
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}
```

사용 예시:

```js
const encoded = await encodeUrl('https://example.com/path?hello=world');
const redirectUrl = `https://r.ameno.cc/?enc=${encoded}`;
console.log(redirectUrl);
```

`CompressionStream` 또는 `DecompressionStream`을 지원하지 않는 오래된 브라우저에서는 `enc` 기능을 사용할 수 없습니다. 기존 `red` 기능은 이 API를 사용하지 않으므로 그대로 동작합니다.
