# Chapter 3 Security and AWS Identity and Access Management (IAM)
## Exercise 3.1
### Up
```sh
aws iam create-user --user-name Alice
```
```sh
aws iam create-user --user-name Bob
```
```sh
aws iam create-user --user-name Charlie
```

### Down
```sh
aws iam delete-user --user-name Alice
```
```sh
aws iam delete-user --user-name Bob
```
```sh
aws iam delete-user --user-name Charlie
```

## Exercise 3.2
### Up
```sh
aws iam create-login-profile --user-name Alice --password # make up a password
aws iam create-access-key --user-name Alice
aws iam create-login-profile --user-name Bob --password # make up another password
```

### Down
```sh
aws iam delete-login-profile --user-name Alice
aws iam delete-access-key --user-name Alice
aws iam delete-login-profile --user-name Bob
```

## Exercise 3.3
(Previous chapter exercises were run)
### Up
```sh
aws iam create-group --group-name admins
aws iam create-group --group-name monitors
aws iam create-group --group-name operators
aws iam add-user-to-group --group-name admins --user-name Alice
aws iam add-user-to-group --group-name monitors --user-name Bob
aws iam add-user-to-group --group-name operators --user-name Charlie
```
### Down
```sh
aws iam remove-user-from-group --group-name admins --user-name Alice
aws iam remove-user-from-group --group-name monitors --user-name Bob
aws iam remove-user-from-group --group-name operators --user-name Charlie
aws iam delete-group --group-name admins
aws iam delete-group --group-name monitors
aws iam delete-group --group-name operators
```
### Info
```sh
aws iam get-group --group-name admins
aws iam get-group --group-name monitors
aws iam get-group --group-name operators
```

