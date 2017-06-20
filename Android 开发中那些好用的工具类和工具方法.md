# Android 开发中那些好用的工具类和工具方法

-----------------------

## Bitmap

- 更换bitmap 的每个像素的颜色

  ```java
  public static Bitmap getColoredBitmap(@NonNull Bitmap bitmap, @ColorInt int color) {
          int alpha = Color.alpha(color);
          int red = Color.red(color);
          int green = Color.green(color);
          int blue = Color.blue(color);

          int[] pixels = new int[bitmap.getWidth() * bitmap.getHeight()];
          bitmap.getPixels(pixels, 0, bitmap.getWidth(), 0, 0, bitmap.getWidth(), bitmap.getHeight());
          for (int i = 0; i < pixels.length; i++) {
              int pixel = pixels[i];
              int pixelAlpha = Color.alpha(pixel);
              if (pixelAlpha != 0)
                  pixels[i] = Color.argb((int) (pixelAlpha * alpha / 256f), red, green, blue);
          }

          Bitmap coloredBitmap = bitmap.copy(Bitmap.Config.ARGB_8888, true);
          coloredBitmap.setPixels(pixels, 0, bitmap.getWidth(), 0, 0, bitmap.getWidth(), bitmap.getHeight());
          return coloredBitmap;
      }
  ```

- 设置bitmap 梯度透明度

  ```java
  public static Bitmap getGradientBitmap(int width, int height, @ColorInt int color) {
          Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);

          int alpha = Color.alpha(color);
          int red = Color.red(color);
          int green = Color.green(color);
          int blue = Color.blue(color);

          int[] pixels = new int[width * height];
          bitmap.getPixels(pixels, 0, width, 0, 0, width, height);
          for (int y = 0; y < height; y++) {
              int gradientAlpha = (int) ((float) alpha * (float) (height - y) * (float) (height - y) / (float) height / (float) height);
              for (int x = 0; x < width; x++) {
                  pixels[x + y * width] = Color.argb(gradientAlpha, red, green, blue);
              }
          }

          bitmap.setPixels(pixels, 0, bitmap.getWidth(), 0, 0, bitmap.getWidth(), bitmap.getHeight());
          return bitmap;
      }
  ```

- 通过`drawableRes` 获取bitmap

  ```java
  public static Bitmap getBitmap(Context context, int drawableRes) {
          Drawable drawable;
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
              drawable = context.getResources().getDrawable(drawableRes, context.getTheme());
          } else {
              drawable = context.getResources().getDrawable(drawableRes);
          }
          Canvas canvas = new Canvas();
          Bitmap bitmap = Bitmap.createBitmap(
                  drawable.getIntrinsicWidth(),
                  drawable.getIntrinsicHeight(),
                  Bitmap.Config.ARGB_8888);
          canvas.setBitmap(bitmap);
          drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
          drawable.draw(canvas);

          return bitmap;
      }
  ```

  ​

## 多次点击监听

```java
public class MultiClickUtil {

    private int clickedTimes = 0;
    private long lastMill = 0;
  
    private final long spaceTime = 2_000;

    public static MultiClickUtil getInstance() {
        return new MultiClickUtil();
    }

    public void setOnMultiClickListener(View view, final int clickTimes,
                                        final OnMultiClickListener listener) {
        if (view == null) {
            return;
        }
        view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (clickTimes <= 0) {
                    return;
                }
                long currentMill = System.currentTimeMillis();
                if (lastMill == 0) {
                    clickedTimes++;
                } else {
                    if (currentMill - lastMill <= spaceTime) {
                        clickedTimes++;
                    } else {
                        clickedTimes = 1;
                    }
                }
                lastMill = currentMill;

                if (clickedTimes >= clickTimes) {
                    listener.onMultiClick(v);
                    clickedTimes = 0;
                    lastMill = 0;
                }
            }
        });
    }

    public interface OnMultiClickListener {
        void onMultiClick(View view);
    }
}
```

