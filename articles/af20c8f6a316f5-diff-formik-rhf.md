---
title: "FormikとReact Hook Formの違いを正しく理解する"
emoji: "🤨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Formik", "ReactHookForm", "React"]
published: false
---

昨今のReact界隈では「FormikのほうがAPIが簡単で優秀だ」「React Hook Form（以下RHF）のほうがAPIがシンプルで使いやすい」などをよく聞くと思います（最近はその勢いも衰えていますが）。

ではなぜそう思うのか、両者の視点から詳しく解説していきます。

# 中立での比較
お馴染みのnpm trendsでは、今の所RHFが優勢のようです。
![](/images/20230413-formik-vs-rhf-npm-trends.png)

> https://npmtrends.com/formik-vs-react-hook-form より引用

2022年6月以前ではFormikが少し追い越していましたが、それからは依然としてRHFが一枚上手です。

# それぞれの特徴

## # Formik
https://formik.org/

Formikは公式サイトで"Build forms in React, without the tears"、要するに「涙を見ずにフォームを作れる」のようなニュアンスの謳い文句があります。
そのAPIはわかりやすく、チュートリアルも見通しが良いためRHFと比べ簡単に見えます（当社比）。

### In-depth
**基本のキ**
とても一般的な、誰でも見たことがあるような書き方ですね。
これについて特筆することはありません。
```jsx
import React from 'react';
import { useFormik } from 'formik';

const Basic = () => {
  const { getFieldProps, handleSubmit } = useFormik({
    initialValues: { name: '' },
    onSubmit: values => {
      console.log(values)
    }
  })

  return (
    <form onSubmit={handleSubmit}>
      <input {...getFieldProps('name')} />
      <button type="submit">Submit</button>
    </form>
  )
}
```

**@mui/x-date-pickers v6 DatePicker**
すでにおかしくなってしまいました。
これはmui側のDatePickerがv5からv6に変わった時の変更も関係しているものの、Formikはこういった特殊なコンポーネントに対する耐性がないため、運コードが生み出されてしまいます（別コンポーネントに切り出すなどやりようはありますが）。
```jsx
import React from 'react';
import { useFormik } from 'formik';
import { DatePicker } from "@mui/x-date-pickers"

const DatePicker = () => {
  const { setFieldValue, setFieldTouched, handleSubmit } = useFormik({
    initialValues: { publishAt: null },
    onSubmit: values => {
      console.log(values)
    }
  })

  return (
    <form onSubmit={handleSubmit}>
      <DatePicker<Date>
        onChange={value => {
          setFieldValue('publishAt', value)
          setFieldTouched('publishAt')
        }}
      />
      <button type="submit">Submit</button>
    </form>
  )
}
```

**タグフィールド**
たいして、Formikは配列に対しての処理がものすごく簡単なため、比較的簡単にタグなどの実装が可能です。
ただし現時点では、配列に関するHooksがないため、少し不便に感じるかもしれません。

```jsx
import React from 'react';
import { Formik, Field, Form, FieldArray } from 'formik';

const initialValues = {
  tags: [
    { label: 'Hoge', value: 'hoge' }
  ]
}

const Tags = () => {
  return (
    <div>
      <Formik
        initialValues={initialValues}
        onSubmit={values => console.log(values)}
      >
        {({ values }) => (
          <Form>
            <FieldArray name="tags">
              {({ push, remove }) => (
                <div>
                  {values.tags.length > 0 &&
                    values.tags.map((tags, index) => (
                      <div>
                        <input 
                          name={`tags.${index}.label`}
                          placeholder="tag label"
                          type="text"
                        />
                        <input 
                          name={`tags.${index}.value`}
                          placeholder="tag value"
                          type="text"
                        />
                        <button
                          type="button"
                          onClick={() => remove(index)}
                        >
                          Remove
                        </button>
                      </div>
                    ))
                  }
                </div>
                <button
                  type="button"
                  onClick={() => push({ label: '', value: '' })}
                >
                  Add tag
                </button>
              )}
            </FieldArray>
          </Form>
        )}
      </Formik>
    </div>
  )
}
```

## # React Hook Form
https://react-hook-form.com/

一方で、RHFは公式サイトで"Performant, flexible and extensible forms with easy-to-use validation."、要するに「パフォーマンスと柔軟性とバリデーションを簡単に作ろう」のようなニュアンスの謳い文句があります。
RHFはFormikと違い、パフォーマンスを前面に出すようなサイトデザインとなっています。レンダー数が少ないなどいろいろありますね。


### In-depth
**基本のキ**
RHFはFormikとか異なり、バリデーションルールをコンポーネント側に定義します。
また、RHFではonSubmit関数をHooksの引数として定義せず、handleSubmitのCallbackとして引数に使います。
これが少し混乱を招くことが多いため、JSにあまり精通していないとここでFormikにいってしまう場合があります（過去の自分）。
```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

const Basic = () => {
  const { register, handleSubmit } = useForm({
    defaultValues: { name: '' },
  })
  const onSubmit = values => console.log(values)

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      <button type="submit">Submit</button>
    </form>
  )
}
```

**@mui/x-date-pickers v6 DatePicker**
Formikに対して、RHFは`Controller`という非常に柔軟なコンポーネント、ないしは`useController`というAPIを持っています。
このControllerがあれば、おそらく対応できないライブラリはないといっても過言ではないでしょう。
そして、こちらもFormikと違い、field.onChangeやfield.onBlurが複雑なCallbackを必要としないため、フレキシブルに合わせることが可能です。

ただし、Controllerというコンポーネントを使用することによって、コードがより長く、複雑化する可能性があります。
```jsx
import React from 'react';
import { useForm, Controller } from 'react-hook-form';
import { DatePicker, LocalizationProvider } from "@mui/x-date-pickers"
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs'

const DatePicker = () => {
  const { handleSubmit, control } = useForm({
    defaultValues: { publishAt: null },
  })
  const onSubmit = values => console.log(values)

  return (
    <LocalizationProvider dateAdapter={AdapterDayjs}>
      <form onSubmit={handleSubmit(onSubmit)}>
        <Controller
          name="publishAt"
          control={control}
          render={({ field: { onChange, onBlur, ...field } }) => 
            <DatePicker<Date>
              {...field}
              onChange={(value) => {
                onChange(value)
                onBlur()
              }}
            />
          }
        />
        <button type="submit">Submit</button>
      </form>
    </LocalizationProvider>
  )
}
```

**タグフィールド**
```jsx

```
