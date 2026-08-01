# Використання окремих SSH-ключів для GitHub та AWS EC2

## Чому краще використовувати різні SSH-ключі

Технічно можна використовувати один і той самий SSH-ключ і для GitHub, і
для AWS EC2. Проте це не рекомендується.

Якщо один приватний ключ буде скомпрометований, зловмисник потенційно
отримає доступ і до GitHub, і до EC2.

## Рекомендована структура

``` text
~/.ssh/
├── github_ed25519
├── github_ed25519.pub
├── aws_ec2_ed25519
└── aws_ec2_ed25519.pub
```

## Створення ключів

### GitHub

``` bash
ssh-keygen -t ed25519 -C "github" -f ~/.ssh/github_ed25519
```

### AWS EC2

``` bash
ssh-keygen -t ed25519 -C "aws-ec2" -f ~/.ssh/aws_ec2_ed25519
```

## Права

``` bash
chmod 600 ~/.ssh/github_ed25519
chmod 600 ~/.ssh/aws_ec2_ed25519
```

## GitHub

Додати ключ до ssh-agent:

``` bash
ssh-add ~/.ssh/github_ed25519
```

Скопіювати публічний ключ:

``` bash
pbcopy < ~/.ssh/github_ed25519.pub
```

Додати ключ у **Settings → SSH and GPG keys → New SSH key**.

Файл `~/.ssh/config`:

``` sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github_ed25519
  IdentitiesOnly yes
```

Перевірка:

``` bash
ssh -T git@github.com
```

## Terraform

``` hcl
resource "aws_key_pair" "ssh_key" {
  key_name   = "terraform-ec2-key"
  public_key = file("~/.ssh/aws_ec2_ed25519.pub")
}

variable "ssh_key_private" {
  default = "/Users/juliadavydova/.ssh/aws_ec2_ed25519"
}
```

## Ansible

``` bash
ansible-playbook   -i inventory_aws_ec2.yaml   --private-key ~/.ssh/aws_ec2_ed25519   --user ec2-user   deploy-docker-new-user.yaml
```

Dynamic inventory:

``` yaml
compose:
  ansible_user: "'ec2-user'"
  ansible_ssh_private_key_file: "'/Users/juliadavydova/.ssh/aws_ec2_ed25519'"
```

## Важливо

Terraform передає в AWS лише **публічний ключ**:

``` text
aws_ec2_ed25519.pub
```

Приватний ключ:

``` text
aws_ec2_ed25519
```

залишається тільки на твоєму комп'ютері.

## Схема

``` text
GitHub:
github_ed25519.pub → GitHub
github_ed25519     → Твій Mac

AWS:
aws_ec2_ed25519.pub → EC2
aws_ec2_ed25519     → Твій Mac / SSH / Ansible
```

## Висновок

Використання окремих SSH-ключів для GitHub та AWS EC2 --- це краща
практика безпеки та більш професійний підхід до DevOps.
