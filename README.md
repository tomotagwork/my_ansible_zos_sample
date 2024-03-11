# Overview

このリポジトリは、Red Hat Ansible Certified Content for IBM Z を使用したAnsible Playbookのサンプルを提供しています。


# 前提

## Ansible control node

- Ansible 2.15.x


## Target z/OS

- z/OS OpenSSH
- IBM Open Enterprise Python for z/OS
- IBM Z Open Automation Utilities


# セットアップ手順

## 1. Ansibleコントロール・ノード上に当リポジトリをクローン

適当なディレクトリで以下のコマンドを実行します。

```sh
git clone https://github.com/tomotagwork/my_ansible_zos_sample.git
```

## 2. 前提となるCollectionをインストール

上で作成されたローカル・リポジトリのディレクトリ下で、以下のコマンドを実行します。

```sh
ansible-galaxy install -r requirements.yml
```

結果としてローカル・リポジトリのディレクトリ下のansible_collections以下に必要なcollectionがインストールされます。

## 3. 接続に関する設定をカスタマイズ

### inventory.yml
inventory.yml を以下のようにカスタマイズします。

```yaml
all:
  children:
    ungrouped: {}
    zos:
      hosts:
        <name>:
          ansible_host: xxx.xxx.xxx.xxx
          ansible_port: 22
          ansible_user: IBMUSER
          ansible_ssh_private_key_file: "~/.ssh/ssh_private_key.pem"
```

補足
- `<name>`: ansible上でハンドリングするターゲット環境(z/OS)の名前
- `ansible_host: xxx.xxx.xxx.xxx`: ターゲット環境(z/OS)のIPアドレスorホスト名
- `ansible_port: 22`: ターゲット環境(z/OS)がListenしているSSHポート番号
- `ansible_user: IBMUSER`: SSH接続ユーザー
- `ansible_ssh_private_key_file: "~/.ssh/ssh_private_key.pem"` : SSH接続ユーザーの認証に使用するPrivate Key

### hostごとの環境変数ファイル

`host_vars/<name>.yml` をカスタマイズします(inventory.ymlで定義した名前と同じ名前のyamlファイルを作成し、その環境用の環境変数を設定します)。

```yaml
PYZ: "/usr/lpp/IBM/cyp/v3r11/pyz"
ZOAU: "/usr/lpp/IBM/zoautil"
ansible_python_interpreter: "{{ PYZ }}/bin/python3"

environment_vars:
  _BPXK_AUTOCVT: "ALL"
  ZOAU_HOME: "{{ ZOAU }}"
  PYTHONPATH: "{{ PYZ }}/lib:{{ ZOAU }}/lib"
  LIBPATH: "{{ ZOAU }}/lib:{{ PYZ }}/lib:/lib:/usr/lib:."
  PATH: "{{ ZOAU }}/bin:{{ PYZ }}/bin:/bin:/var/bin"
  _CEE_RUNOPTS: "FILETAG(AUTOCVT,AUTOTAG) POSIX(ON)"
  _TAG_REDIR_ERR: "txt"
  _TAG_REDIR_IN: "txt"
  _TAG_REDIR_OUT: "txt"
  LANG: "C"
  _BPX_SHAREAS: "YES"
  _BPX_BATCH_SPAWN: "YES"
```

補足
- `PYZ: "/usr/lpp/IBM/cyp/v3r11/pyz"`: ターゲット環境(z/OS)のPythonのインストール・パス
- `ZOAU: "/usr/lpp/IBM/zoautil"`: ターゲット環境(z/OS)のZOAUのインストール・パス
- `ansible_python_interpreter: "{{ PYZ }}/bin/python3"`: ターゲット環境(z/OS)のPythonプログラムのパス

### Playbook (zos_operator.yml)

```yaml
- name: submit a zos command
  hosts: <name>
  environment: "{{ environment_vars }}"
  gather_facts: no
  ...
```

inventoryに指定した管理対象のホスト名に合わせてPlaybook中のhosts指定を変更します。


## 4. サンプルのPlaybookを実行(稼働確認)

ローカル・リポジトリのディレクトリで以下のコマンドを実行し、サンプルのPlaybook "zos_operator.yml"を実行。

```sh
ansible-playbook sample/zos_operator.yml 
```

※MVSコマンド `D T` (時刻を取得するコマンド)を実行し、その結果を出力するだけの単純なPlaybook

結果サンプル
```
user01@IBM-PF3ALW3Q:~/Ansible/VSCode_workspace/my_ansible_zos_sample$ ansible-playbook sample/zos_operator.yml

PLAY [submit a zos command] *********************************************************************************************************************************************

TASK [envvars] **********************************************************************************************************************************************************
ok: [ezdwazi04] => {
    "msg": "environemt_variables are {'_BPXK_AUTOCVT': 'ALL', 'ZOAU_HOME': '/usr/lpp/IBM/zoautil', 'PYTHONPATH': '/usr/lpp/IBM/cyp/v3r11/pyz/lib', 'LIBPATH': '/usr/lpp/IBM/zoautil/lib:/usr/lpp/IBM/cyp/v3r11/pyz/lib:/lib:/usr/lib:.', 'PATH': '/usr/lpp/IBM/zoautil/bin:/usr/lpp/IBM/cyp/v3r11/pyz/bin:/bin:/var/bin', '_CEE_RUNOPTS': 'FILETAG(AUTOCVT,AUTOTAG) POSIX(ON)', '_TAG_REDIR_ERR': 'txt', '_TAG_REDIR_IN': 'txt', '_TAG_REDIR_OUT': 'txt', 'LANG': 'C', '_BPX_SHAREAS': 'YES', '_BPX_BATCH_SPAWN': 'YES'}"
}

TASK [execute an operator command] **************************************************************************************************************************************
ok: [ezdwazi04]

TASK [display results] **************************************************************************************************************************************************
ok: [ezdwazi04] => {
    "msg": "command output {'changed': False, 'rc': 0, 'elapsed': 1.02, 'content': ['VS01       2024070  07:06:07.00             ISF031I CONSOLE IBMU0000 ACTIVATED', 'VS01       2024070  07:06:07.00            -D T ', 'VS01       2024070  07:06:07.00             IEE136I LOCAL: TIME=07.06.06 DATE=2024.070  UTC: TIME=11.06.06 DATE=2024.070'], 'cmd': 'D T', 'wait_time_s': 1, 'failed': False}"
}

PLAY RECAP **************************************************************************************************************************************************************
ezdwazi04                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
