---
layout:     post
title:      UIAlertController修改标题和按钮颜色
subtitle:   iOS技术成长之路
date:       2020-09-22
author:     DM
header-img: img/post-bg-debug.png
catalog: true
tags: iOS   
---


## UIAlertController 修改标题 和按钮颜色

```
	UIAlertController *actionSheet = [UIAlertController alertControllerWithTitle:@"" message:@"您确定要取消支付吗？" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *action1 = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
    UIAlertAction *action2 = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action)
    {
        
        [self.navigationController popViewControllerAnimated:YES];
    }];
    //设置字体颜色
    [action1 setValue:RGB(204, 204, 204) forKey:@"titleTextColor"];
    [action2 setValue:kMainColor forKey:@"titleTextColor"];
    
    NSMutableAttributedString *hogan1 = [[NSMutableAttributedString alloc] initWithString:@"您确定要取消支付吗？"];
    [hogan1 addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:16] range:NSMakeRange(0, 10)];
    [actionSheet setValue:hogan1 forKey:@"attributedMessage"];
    //把action添加到actionSheet里
    [actionSheet addAction:action1];
    [actionSheet addAction:action2];

    //相当于之前的[actionSheet show];
    [self presentViewController:actionSheet animated:YES completion:nil];
    
    ```
