# how_to_make_original_command_with_gitbash
Windows の GitBash で実行できるオリジナルコマンドを作成する方法

## 具体例

以下のオリジナルコマンド `getdiff` が実行できるようにする。

```
getdiff 引数
```

getdiff コマンドは、以下のようなコマンドを実行すると、対象ディレクトリの `layout__.tsx` と `layout.tsx` の差分をコンソールに出力する。

```
getdiff ./src/app/workspace/\[workspaceId\]/layout.tsx
```

### scripts フォルダの作成

以下のコマンドを実行して、ユーザフォルダに `scripts` フォルダを作成する。

```
mkdir ~/scripts
```

Windows では以下の場所にフォルダが作成される。

Bash 式のパス記載
```
/c/Users/<YourUsername>/scripts
```

Windows 式のパス記載
```
C:\Users\<YourUsername>\scripts
```

### Git Bash の設定ファイルに追記

Git Bash の設定ファイル（~/.bashrc  # または ~/.bash_profile）に以下の記述を追記する。

```
chcp.com 65001
export PATH="$PATH:/c/Users/<YourUsername>/scripts"
alias getdiff='/c/Users/hsmt7/scripts/getdiff.sh'
```

設定ファイルに `export PATH="$PATH:<DirectoryPath>"` を追加することで、対象ディレクトリのファイルがフルパスではなく、直接ファイル名を指定して実行することができる。

設定ファイルにエイリアスを追加することで、getdiff コマンドが .sh を指定せずに実行できるようになる。

### ファイルの配置

エイリアスでしたシェルのファイルを配置する。

例えば、`~/scripts` に配置する `getdiff.sh` ファイルの内容は以下のようにする。

C:\Users\<YourUsername>\scripts\getdiff.sh
```
#!/bin/bash

# コンソールをクリアする
clear

# 引数で渡されたファイルパスを受け取る
file_path="$1"

# ファイルパスが空でないことを確認
if [ -z "$file_path" ]; then
  echo "Usage: getdiff <file_path>"
  exit 1
fi

# 拡張子を取り除いたファイル名部分を取得
base_name=$(basename "$file_path" | sed 's/\(.*\)\..*/\1/')

# 元のファイルと新しいファイル名を設定
file1="${file_path%/*}/$base_name"__.${file_path##*.}  # 拡張子の前に '__' を追加
file2="$file_path"  # 元のファイル

# ファイルの存在チェック
if [ ! -f "$file1" ]; then
  echo "Error: File '$file1' not found!"
  exit 1
fi

if [ ! -f "$file2" ]; then
  echo "Error: File '$file2' not found!"
  exit 1
fi

# diff コマンドを実行
echo "Running diff between $file1 and $file2"
diff -U1000 "$file1" "$file2"
```

### コマンド実行結果の出力例

``` diff
 'use client';
 
 import { Loader } from 'lucide-react';
 
 import {
   ResizableHandle,
   ResizablePanel,
   ResizablePanelGroup,
 } from '@/components/ui/resizable';
 import { Profile } from '@/features/members/components/profile';
 import { Thread } from '@/features/messages/components/thread';
 import { usePanel } from '@/hooks/use-panel';
 
 import { Sidebar } from './sidebar';
 import Toolbar from './toolbar';
 import { WorkspaceSidebar } from './workspace-sidebar';
 
 import type { Id } from '../../../../convex/_generated/dataModel';

 interface WorkspaceLayoutProps {
   children: React.ReactNode;
 }

 const WorkspaceLayout = ({ children }: WorkspaceLayoutProps) => {
   const { profileMemberId, parentMessageId, onClose } = usePanel();

   const isShowPanel = !!profileMemberId || !!parentMessageId;

   return (
     <div className="h-full">
       <Toolbar />
       <div className="flex h-[calc(100vh-40px)]">
         <Sidebar />
         <ResizablePanelGroup
           direction="horizontal"
           autoSaveId="workspace-layout-setting"
         >
           <ResizablePanel
             defaultSize={20}
             minSize={11}
             className="bg-[#2e325e]"
           >
             <WorkspaceSidebar />
           </ResizablePanel>
           <ResizableHandle withHandle />
-          <ResizablePanel minSize={20}>{children}</ResizablePanel>
+          <ResizablePanel
+            minSize={20}
+            defaultSize={80}
+          >
+            {children}
+          </ResizablePanel>
           {isShowPanel && (
             <>
               <ResizableHandle withHandle />
               <ResizablePanel
                 minSize={20}
                 defaultSize={29}
               >
                 {parentMessageId ? (
                   <Thread
                     messageId={parentMessageId as Id<'messages'>}
                     onClose={onClose}
                   />
                 ) : profileMemberId ? (
                   <Profile
                     memberId={profileMemberId as Id<'members'>}
                     onClose={onClose}
                   />
                 ) : (
                   <div className="flex h-full items-center justify-center">
                     <Loader className="size-5 animate-spin text-muted-foreground" />
                   </div>
                 )}
               </ResizablePanel>
             </>
           )}
         </ResizablePanelGroup>
       </div>
     </div>
   );
 };

 export default WorkspaceLayout;
```

以上。
