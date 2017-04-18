手动操作是可选的和可阻止的，阻止手动操作将会在操作定义pipeline阶段执行。当有人通过按钮执行阻止手动操作切换时候，可以恢复pipeline的执行。

当pipeline构建成功，但是pipeline 被阻止的时候提交合并则不会合并Pipeline，pipeline也有一种特殊的状态叫做 manual

manual操作默认是不阻止的，如果你想让manual操作阻止，就增加`allow_failure:false`添加到`.gitlab-ci.yml`.`allow_failure: true`默认情况下，可选手动操作已设置。


