(Previous chapter exercises were run)
### Up
```
aws iam create-group --group-name admins
aws iam create-group --group-name monitors
aws iam create-group --group-name operators
aws iam add-user-to-group --group-name admins --user-name Alice
aws iam add-user-to-group --group-name monitors --user-name Bob
aws iam add-user-to-group --group-name operators --user-name Charlie
```
### Down
```
aws iam remove-user-from-group --group-name admins --user-name Alice
aws iam remove-user-from-group --group-name monitors --user-name Bob
aws iam remove-user-from-group --group-name operators --user-name Charlie
aws iam delete-group --group-name admins
aws iam delete-group --group-name monitors
aws iam delete-group --group-name operators
```
### Info
```
aws iam get-group --group-name admins
aws iam get-group --group-name monitors
aws iam get-group --group-name operators
```

