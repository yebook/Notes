### [LiveWallpaper: java.lang.IllegalStateException: Surface has already been released](https://stackoverflow.com/questions/20665636/livewallpaper-java-lang-illegalstateexception-surface-has-already-been-release)

```java
@Override
    public void onSurfaceCreated(SurfaceHolder holder) {
        super.onSurfaceCreated(holder);
        handler.post(reload);
        drawOk = true;
    }

    @Override
    public void onSurfaceDestroyed(SurfaceHolder holder) {
        super.onSurfaceDestroyed(holder);
        handler.removeCallbacks(reload);
        drawOk = false;
    }

    @Override
    public void onVisibilityChanged(boolean visible) {
        super.onVisibilityChanged(visible);
        if(visible) {
            handler.post(reload);
            drawOk = true;
        } else {
            handler.removeCallbacks(reload);
            drawOk = false;
        }
    }

    private void draw() {

        SurfaceHolder surfaceHolder = getSurfaceHolder();
        Canvas canvas = null;

        if(drawOk) {
            canvas = surfaceHolder.lockCanvas();
            if(canvas != null) {
                                // ...
            }
        }
    }   
```

