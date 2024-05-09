---
layout: post
title: "Làm thế nào để override 3rd typescript types được khai báo bởi install package trong node modules"
date: 2024-05-09 09:00:00 +0700
categories: Tips
tags:
  - Typescript
hide_thumbnail: true

---

Hiện tại mình đang sử dụng thư viện [openai-node](https://github.com/openai/openai-node). Tuy nhiên khi implement code theo sample trong Document ([Uploading Your Batch Input File](https://platform.openai.com/docs/guides/batch/2-uploading-your-batch-input-file)) của thư viện thì phát sinh lỗi:

```javascript
TSError: ⨯ Unable to compile TypeScript:
worker/generate.ts:18:4 - error TS2322: Type '"batch"' is not assignable to type '"fine-tune" | "assistants"'.

18    purpose: "batch",
      ~~~~~~~

  node_modules/openai/resources/files.d.ts:119:5
    119     purpose: 'fine-tune' | 'assistants';
            ~~~~~~~
    The expected type comes from property 'purpose' which is declared here on type 'FileCreateParams'
```

Đi vào trong source code của node_modules pakage này mình phát hiện một điều, trong file [openai-node/src/resources/files.ts - L185](https://github.com/openai/openai-node/blob/7196ac9310d58d057fb2a575e60c1718bf6341a2/src/resources/files.ts#L185). Property `purpose` thiếu mất `type` là **batch**.

```javascript
purpose: 'fine-tune' | 'assistants';
```

Trong thời gian chờ team dev của OpenAI fix lỗi này (Mình đã log 1 issue tại [đây](https://github.com/openai/openai-node/issues/830)). Để có thể dùng được thư viện, chúng ta cần override lại **FileCreateParams Interface**.

Có khá nhiều cách, trong bài viết này mình sử dụng `Remap paths trong tsconfig`. Theo nghĩa đen, chỉ cần sao chép, dán nguyên văn nội dung file từ node_modules và áp dụng các bản sửa lỗi của riêng bạn.


```json
// tsconfig.json
{
  "paths": {
    /* https://stackoverflow.com/questions/67701504/best-way-to-override-third-party-typescript-types-in-node-modules */
    "openai/resources/*": ["worker/declarations/openai/resources/*"]
  }
}
```

Trong thư mục `worker/declarations/openai/resources/` mình tạo 1 file `index.d.ts` với nội dung sửa lại `purpose` như sau:

```typescript
...

export interface FileCreateParams {
	/**
	 * The File object (not file name) to be uploaded.
	 */
	file: Uploadable;
	/**
	 * The intended purpose of the uploaded file.
	 *
	 * Use "fine-tune" for
	 * [Fine-tuning](https://platform.openai.com/docs/api-reference/fine-tuning) and
	 * "assistants" for
	 * [Assistants](https://platform.openai.com/docs/api-reference/assistants) and
	 * [Messages](https://platform.openai.com/docs/api-reference/messages). This allows
	 * us to validate the format of the uploaded file is correct for fine-tuning.
	 */
	purpose: "assistants" | "batch" | "fine-tune";
}
export interface FileListParams {
	/**
	 * Only return files with the given purpose.
	 */
	purpose?: string;
}

...
//# sourceMappingURL=files.d.ts.map

```

Cảm ơn bạn đã quan tâm tới tận cuối bài viết 😆
