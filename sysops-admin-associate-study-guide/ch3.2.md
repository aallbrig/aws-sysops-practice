### Up
```
aws iam create-login-profile --user-name Alice --password # make up a password
aws iam create-access-key --user-name Alice
aws iam create-login-profile --user-name Bob --password # make up another password
```

### Down
```
aws iam delete-login-profile --user-name Alice
aws iam delete-access-key --user-name Alice
aws iam delete-login-profile --user-name Bob
```
