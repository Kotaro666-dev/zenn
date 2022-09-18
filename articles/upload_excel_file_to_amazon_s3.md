---
title: "ExcelファイルをAmazonS3へアップロードする/からダウンロードする【Node.js】"
emoji: "🐡"
type: "tech"
topics: ["Node", "s3", "AWS", "typescript"]
published: true
---

# 目的

APIを通じて、フロントエンドからバックエンドに Excel ファイルのデータが送られてきます。
送られてきた Excel ファイルのデータを、Amazon S3(Simple Storage Service)にファイルとしてアップロードします。
また、フロントエンド側でファイルダウンロードイベント実行時に、Amazon S3 に保存された Excel ファイルのデータをバックエンドで読み取り、フロントエンドにデータを返します。

# 背景

業務内で開発しているアプリケーション内で、クライアントがExcelファイルを保存する機能の要望がありました。

# 特記事項

1. フロントエンドから送られてくる Excel ファイルのデータは、 Base64 方式にエンコードされている
2. フロントエンドに返す Excel ファイルのデータは、Base64 方式が期待されている
3. 対応する拡張子ファイルは、.xls, .xlsx, .xlsm の 3 つである

# 使用フレームワーク/ライブラリ等

- バックエンド
  - Next.js
  - Node.js
  - @aws-sdk/client-s3

# 開発環境

```
$ npm info next version
12.3.0

$ node -v
v16.15.1

$ tsc -v
Version 4.7.3

$ npm info @aws-sdk/client-s3 version
3.171.0
```

# S3 クライアントのインスタンスを作成する

環境ごとに S3 クライアントのインスタンス作成時に引数として渡す設定情報を変えています。

本番環境では、アプリケーションが AWS EC2 にデプロイされており、S3 へのアクセスを許可する AWS IAMプロファイルロールがEC2にアタッチされているため、それを通じて認証されます。

開発環境では、ローカルホストを使っているため、認証情報を明示的に渡すことで、S3 との認証を行います。

※本番環境用でも region 情報を渡さない場合には認証エラーが発生していたため、インスンスの設定情報には必ず region 情報を渡す必要がありそうです。

```typescript
class ExcelRepository {
    private s3Client: S3Client;

    public constructor() {
        if (process.env.NODE_ENV === 'development') {
            this.s3Client = new S3Client({
                region: 'XXX-XXXXXXX-X',
                credentials: {
                    accessKeyId: 'XXXXXXXXXXXXXXXXXXXXXXXX',
                    secretAccessKey: 'XXXXXXXXXXXXXXXXXXXXXXXX',
                },
            });
        } else if (process.env.NODE_ENV === 'production') {
            this.s3Client = new S3Client({
                region: 'XXX-XXXXXXX-X',
            });
        }
    }
}
```

# S3 にファイルをアップロードする

S3 にファイルをアップロードする基本的なコードは、[こちらの公式ガイド](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/s3-example-creating-buckets.html#s3-example-creating-buckets-upload-file)をご覧ください。

Base64にエンコードされたデータを `Buffer` を使って、バイナリデータに変換します。

PutObjectCommandInput.Body は型として、`string | Uint8Array | Buffer` を許容するため、Buffer型で渡して問題ありません（[参考リンク](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/interfaces/putobjectcommandinput.html)）。

```typescript
public async uploadFile(
        fileName: string,
        fileContents: string,
        contentType: string,
    ): Promise<PutObjectOutput | Error> {
        try {
            const parameters = {
                Bucket: 'XXXXXXXXXXXXXXXXXXXXXXXX',
                Key: fileName,
                Body: this.convertFileContentsFromBase64ToBinary(fileContents),
                ContentType: contentType, // 例: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
            };
            return await this.s3Client.send(new PutObjectCommand(parameters));
        } catch (error) {
            return error;
        }
    }

private static convertFileContentsFromBase64ToBinary(fileContents: string): Buffer {
        return Buffer.from(fileContents, 'base64');
    }
```

# S3 からファイルをダウンロードする

S3 からファイルをダウンロードするする基本的なコードは、[こちらの公式ガイド](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/s3-example-creating-buckets.html#s3-example-creating-buckets-get-object
)をご覧ください。

S3からファイルをダウンロードしたレスポンスのうち、ファイルデータは `Body` に格納されています。
しかしながら、SDK V3 から `Body` の型は、`Readable | ReadableStream | Blob` であり、
base64 にエンコードされた string 型に変換するためには特別な処理が必要です。

同じ問題の対応方法について議論していた[Github Issue](https://github.com/aws/aws-sdk-js-v3/issues/1877#issuecomment-755430937) を参考にして実装しました。
今回の場合には、Base64 にエンコードしたデータをフロントエンドに返したいため、`toString('base64')` という処理を行います。

```typescript
public async downloadFile(
    objectKey: string,
): Promise<string | Error> {
    try {
        const parameters = {
            Bucket: process.env.AWS_S3_BUCKET_NAME,
            Key: objectKey,
        };
        const result = await this.s3Client.send(new GetObjectCommand(parameters));
        return await this.streamToString(
            result.Body as Readable,
            'base64',
        );
    } catch (error) {
        return error;
    }
}

// 参考: https://github.com/aws/aws-sdk-js-v3/issues/1877#issuecomment-755430937
private async streamToString(
    stream: Readable,
    bufferEncoding: BufferEncoding,
): Promise<string> {
    return await new Promise((resolve, reject) => {
        const chunks: Uint8Array[] = [];
        stream.on('data', (chunk) => chunks.push(chunk));
        stream.on('error', reject);
        stream.on('end', () =>
            resolve(Buffer.concat(chunks).toString(bufferEncoding)),
        );
    });
}
```