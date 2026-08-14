<script>
function applyAndroidSystemUi() {
  // #ifdef APP-PLUS
  if (plus.os.name !== 'Android') return

  try {
    const activity = plus.android.runtimeMainActivity()
    const window = plus.android.invoke(activity, 'getWindow')
    const decorView = plus.android.invoke(window, 'getDecorView')
    const Color = plus.android.importClass('android.graphics.Color')
    const BuildVersion = plus.android.importClass('android.os.Build$VERSION')
    const transparent = Color.parseColor('#00000000')

    plus.android.invoke(window, 'setNavigationBarColor', transparent)

    if (BuildVersion.SDK_INT >= 26) {
      const LAYOUT_STABLE = 0x100
      const LAYOUT_HIDE_NAVIGATION = 0x200
      const LAYOUT_FULLSCREEN = 0x400
      const LIGHT_NAVIGATION_BAR = 0x10
      const currentFlags = plus.android.invoke(decorView, 'getSystemUiVisibility') || 0
      plus.android.invoke(
        decorView,
        'setSystemUiVisibility',
        currentFlags | LAYOUT_STABLE | LAYOUT_HIDE_NAVIGATION | LAYOUT_FULLSCREEN | LIGHT_NAVIGATION_BAR
      )
    }

    if (BuildVersion.SDK_INT >= 28) {
      plus.android.invoke(window, 'setNavigationBarDividerColor', transparent)
    }

    if (BuildVersion.SDK_INT >= 29) {
      plus.android.invoke(window, 'setNavigationBarContrastEnforced', false)
    }

    // Android 11+ 需要关闭 DecorView 对系统栏的自动避让，页面背景才能绘制到
    // 手势导航条后面。否则旧 targetSdk 的运行基座仍会保留一整块黑色区域。
    if (BuildVersion.SDK_INT >= 30) {
      plus.android.invoke(window, 'setDecorFitsSystemWindows', false)
    }
    console.log('[system-ui] Android edge-to-edge applied')
  } catch (error) {
    console.error('[system-ui] Android edge-to-edge failed', error)
  }
  // #endif
}

export default {
  onLaunch: function () {
    console.log('App Launch')
    applyAndroidSystemUi()
  },
  onShow: function () {
    console.log('App Show')
    // Activity 从后台恢复时 Android 可能重置系统 UI 状态。
    setTimeout(applyAndroidSystemUi, 100)
  },
  onHide: function () {
    console.log('App Hide')
  },
}
</script>

<style>
/*每个页面公共css */
</style>
