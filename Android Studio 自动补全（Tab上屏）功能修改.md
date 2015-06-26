# Android Studio 自动补全（Tab上屏）功能修改

Android Studio都发布了好几个版本了，但是团队的很多项目还是用eclipse。其实个人也用过AS一段时间，开发体验很好所以转向AS也是大势所趋。特别是自动补全功能，和sublime一样用着很顺畅。常用的功能快键基本上都能与eclipse对得上，配上VIM插件编码时很有流输入的感觉。

之前看到有人分享怎么修改eclipse自动补全上屏功能文章觉得很实用，所以就想在AS上也修改修改。其实AS提供了好几种补全上屏的方式，只是不太符合我的习惯。准确的说我是想把Tab上屏功能改成上下变换选择项而已。就像输入法一样按Tab或都Shift+Tab来回选择需要输入的词，而不用找向上向下箭头。

#### 具体操作：
- #####　下载编译器和源码
下载[intellij IDEA Community Edition](https://www.jetbrains.com/idea/download/)和 Android Studio对应的[源码包](http://www.jetbrains.org/display/IJOS/Download)。Android Studio与intellij IDEA的版本可以在[Android Tools官网](http://tools.android.com/build)查看。

- #####    修改和编译
在IDEA中打开源码工程，找到platform/lang-impl/src/com/intellij/codeInsight/lookup/impl/actions/ChooseItemAction.java 修改并编译工程。修改ChooseItemAction.java中的两个函数：
#

>
    @Override
    public void execute(@NotNull final Editor editor, final DataContext dataContext) {
      final LookupImpl lookup = (LookupImpl)LookupManager.getActiveLookup(editor);
      boolean isfinished =true;
      if (lookup == null) {
        throw new AssertionError("The last lookup disposed at: " + LookupImpl.getLastLookupDisposeTrace() + "\n-----------------------\n");
      }
      if (finishingChar == Lookup.NORMAL_SELECT_CHAR) {
        if (!lookup.isFocused()) {
          FeatureUsageTracker.getInstance().triggerFeatureUsed(CodeCompletionFeatures.EDITING_COMPLETION_CONTROL_ENTER);
        }
      } else if (finishingChar == Lookup.COMPLETE_STATEMENT_SELECT_CHAR) {
        FeatureUsageTracker.getInstance().triggerFeatureUsed(CodeCompletionFeatures.EDITING_COMPLETION_FINISH_BY_SMART_ENTER);
      } else if (finishingChar == Lookup.REPLACE_SELECT_CHAR) {
        if (focusedOnly)
        {
          ListScrollingUtil.moveUp(lookup.getList(), 0);
        }else {
          ListScrollingUtil.moveDown(lookup.getList(), 0);
        }
        lookup.markSelectionTouched();
        lookup.refreshUi(false, true);
        lookup.setFocusDegree(LookupImpl.FocusDegree.FOCUSED);
        isfinished = false;
      } else if (finishingChar == '.')  {
        FeatureUsageTracker.getInstance().triggerFeatureUsed(CodeCompletionFeatures.EDITING_COMPLETION_FINISH_BY_CONTROL_DOT);
      }
      if (isfinished)
      lookup.finishLookup(finishingChar);
    }
>
    public static class ChooseWithDot extends ChooseItemAction {
        public ChooseWithDot() {
            super(new Handler(true, Lookup.REPLACE_SELECT_CHAR));
        }
    }

- ##### 替换Android Studio文件
在Android Studio安装目录下找到lib/idea.jar包。将上一步编译生成的class文件（out/production/lang-impl/com/intellij/codeInsight/lookup/impl/actions/）替换掉idea.jar包中对应的文件 (/com/intellij/codeInsight/lookup/impl/actions/)。修改前请做好备份。

- ##### 修改快捷键
修改Android Studio中 EditorChooseLookupItemDot 的快捷键为SHIFT+TAB。
