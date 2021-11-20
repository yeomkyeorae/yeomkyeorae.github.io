---
title: "Node로 AWS S3에 이미지 업로드(jpeg, heic) 및 삭제하기"
excerpt: "NodeJS, AWS S3 활용 이미지 업로드 및 삭제"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - deploy
tags:
  - deploy
  - aws
  - nodejs
  - mongodb
---



저번에 Heroku를 이용해 `푸드마커` 토이 프로젝트를 배포해본 것에 이어 AWS S3를 이용해 이미지를 업로드했던 과정을 적어보겠습니다

S3 버킷 생성, S3 옵션 설정, IAM 사용자 생성, 엑세스 키 생성 등의 내용은 생략되어 있습니다.



## 1. 패키지 설치 및 인스턴스 생성

먼저 S3에 이미지를 업로드 하기 위한 패키지 2가지를 npm을 활용해 install 합니다.

```bash
$ npm install --save aws-sdk multer-s3
```

그리고 아래와 같이 인스턴스를 생성해 줍니다.

저는 `process.env.NODE_ENV`값이 production인 경우에만 패키지를 가져오도록 했습니다.

그리고 `env` 값에 정의한 `AWSAccessKeyId`와 `AWSSecretKey`와 S3 버킷을 생성할 때의 `region`을 기입해 AWS config를 업데이트 합니다.

그리고 class 인스턴스를 생성할 때 처럼 new 키워드를 사용해 AWS 인스턴스를 생성합니다.

```javascript
// when deploy
let upload;	        // jpeg 이미지 업로드용 함수
let uploadHeic;		// heic 이미지 업로드용 함수
let deleteImages;	// 이미지 삭제용 함수
if(process.env.NODE_ENV === 'production') {
  const multerS3 = require('multer-s3');
  const AWS = require('aws-sdk');

  AWS.config.update({
    accessKeyId: process.env.AWSAccessKeyId,
    secretAccessKey: process.env.AWSSecretKey,
    region: 'ap-northeast-2',
  });

  const s3 = new AWS.S3();
  
  // jpeg 이미지 업로드용 함수
  
  // heic 이미지 업로드용 함수
  
  // 이미지 삭제용 함수
}
```

제가 AWS S3를 활용해 이미지를 다룬 내용은 총 3가지 입니다. `Jpeg 이미지의 업로드`, `heic 이미지의 업로드` 그리고 `업로드한 이미지의 삭제`입니다.

차례대로 아래에서 확인해 보겠습니다.



## 2. jpeg 이미지 업로드

jpeg 이미지는  `form data`로 POST request를 보내므로  `multerS3`를 활용했습니다

기존 `multer`에서 사용하는 방식과 유사합니다. s3 인스턴스와 bucket을 object로 넣어 줍니다.

제 토이 프로젝트에서는 여러 개의 이미지를 업로드 할 수 있으므로, `array` 메소드를 호출해 10개까지의 이미지를 업로드 할 수 있도록 했습니다.

```javascript
// jpeg 이미지 업로드용 함수
upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: 'food-marker',
    key(req, file, cb) {
      //uploads 경로에 이미지 업로드
      cb(null, `uploads/${Date.now()}_${file.originalname}`);
    },
  }),
  limits: { fileSize: 20 * 1024 * 1024 },
}).array("restaurant_jpeg_img", 10);
```

jpeg을 업로드할 수 있는 post 함수를 다음과 같이 작성했습니다.

정상적으로 S3에 업로드 되면 `location` 이름으로 이미지 파일이 저장된 `url`를 반환합니다. 

```javascript
// post jpeg img
app.post("/api/img/jpeg", (req, res) => {
  upload(req, res, err => {
    if (err) {
      return res.json({ success: false, err });
    }
    const fileNames = process.env.NODE_ENV === 'production' ? res.req.files.map(file => file.location)
      : res.req.files.map(file => `http://localhost:5000/food/` + file.filename);

    return res.json({ success: true, fileNames: fileNames });
  });
});
```



## 3. heic 이미지 업로드

제 프로젝트에서 heic 이미지는 클라이언트에서 jpeg으로 변환되는데,  jpeg 이미지와 달리 encoding 형식이 `base64`입니다. 

따라서 multerS3를 사용하지 않고 바로 s3 인스턴스의 `upload` 메소드를 이용했습니다. 인자에 `Bucket`, `ContentEncoding`, `ContentType` 등을 기입해 주었습니다.

함수 반환 값으로 `Promise`객체를 반환해야 했기 때문에 `.promise()` 메소드를 호출합니다. 결과 값에 `Location`에 저장된 이미지의 `url`이 담겨 있습니다. 대문자로 시작하는 것에 유의해야 합니다.

```javascript
// heic 이미지 업로드용 함수
uploadHeic = async (heicFile, fileName) => {
  const params = {
    Bucket: 'food-marker',
    Key: fileName,
    Body: heicFile,
    ContentEncoding: 'base64',
    ContentType: 'image/jpeg'
  }

  try {
    const result = await s3.upload(params).promise();
    return result.Location;
  } catch(err) {
    console.log(err);
  }
}
```

heic 이미지를 업로드할 수 있는 post 함수는 아래와 같이 작성했습니다.

`Buffer.from` 으로 buffer를 생성해 `uploadHeic` 함수에 넘겨주는 형식입니다.

heic 파일 하나 하나를 `uploadHeic` 함수를 호출해 비동기 처리를 해야 했기 때문에 `Promise.all`를 사용하게 되었습니다.

아직까지 `Promise` 객체를 사용하는 것이 어색하기만 합니다.

```javascript
// post heic img
app.post("/api/img/heic", async (req, res) => {
  const images = req.body.images;
  const imageNames = req.body.imgNames;

  if(process.env.NODE_ENV === 'production') {
      try {
        const heicImagePaths = [];

        await Promise.all(imageNames.map(async (imageName, ix) => {
          const img = images[ix].replace(/^data:image\/jpeg;base64,/, "");	// 불필요한 부분 제거
          const buf = Buffer.from(img, 'base64');
          const imgFullName = `uploads/${Date.now()}_${imageName}.jpeg`;

          const imgClientPath = await uploadHeic(buf, imgFullName);
          heicImagePaths.push(imgClientPath);
        }))
        return res.json({ success: true, fileNames: heicImagePaths });
      } catch(err) {
        return res.json({ success: false, err });
      }
  }
  // ...
});
```



## 4. 업로드 이미지 삭제

이미지 삭제의 경우에는 s3 인스턴스의 `deleteObjects` 메소드를 호출합니다. 여러 개의 이미지를 업로드할 수 있으므로, 마찬가지로 여러 개의 이미지를 한꺼번에 삭제할 수 있습니다. 인자에 `Bucket`,  `Delete`를  객체 형식으로 넣어줍니다. `Delete`에는 `Objects`가 포함돼 있습니다. `Objects`는 배열 형식으로  `{Key: '삭제할 S3 이미지 경로'}` 값을 갖도록 했습니다. 마찬가지로 `.promise()` 메소드를 호출해 반환합니다.

```javascript
// 이미지 삭제용 함수
deleteImages = async objectArr => {
  const params = {
    Bucket: "food-marker", 
    Delete: {
      Objects: objectArr, 
      Quiet: false
    }
  };

  try {
    const result = await s3.deleteObjects(params).promise();
    return result;
  } catch(err) {
    console.log(err);
  }
}
```

이미지를 삭제할 delete 함수는 아래와 같이 작성했습니다. `deleteImages` 함수에 인자로 `objectArr`라는 이름의 배열을 넘겨주었습니다. 이 배열 요소에는 삭제할 이미지 경로를 `Key` 에 할당한 객체가 담겨있습니다.

```javascript
// delete my restaurant
app.delete("/api/restaurant", (req, res) => {
  Restaurant.findOneAndRemove({ _id: req.query._id }, async (err, restaurantInfo) => {
    if (err)
      return res.json({
        success: false,
        err
      });

    const restaurantImgURLs = restaurantInfo.imgURL.split(",");
    if(process.env.NODE_ENV === 'production') {
      try {
        const objectArr = restaurantImgURLs.map(url => {
          const splited = url.split('uploads');
          const key = 'uploads' + splited[splited.length - 1];
          return { Key: key }
        })
        const result = await deleteImages(objectArr);

        return res.json({ success: true });
      } catch(err) {
        return res.json({ success: false, err });
      }
    }
    // ...
});
```

다음에는 AWS lambda를 활용해 이미지 사이즈를 줄인 내용으로 글을 작성해 보도록 하겠습니다. 읽어주셔서 감사합니다!



## 참고자료

1. [Heroku에서 AWS S3 사용하기](https://velog.io/@gbskang/Heroku%EC%97%90%EC%84%9C-AWS-S3-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
2. [NODEJS AWS의 S3에 파일 업로드 다운로드](https://brunch.co.kr/@daniellim/43)
3. [Nodejs aws-sdk S3 오브젝트 삭제](https://velog.io/@nawnoes/Nodejs-aws-sdk-S3-%EC%98%A4%EB%B8%8C%EC%A0%9D%ED%8A%B8-%EC%82%AD%EC%A0%9C)
4. [How to synchronously upload files to S3 using aws-sdk?](https://stackoverflow.com/questions/57420576/how-to-synchronously-upload-files-to-s3-using-aws-sdk)
5. [https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#deleteObjects-property](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#deleteObjects-property)



