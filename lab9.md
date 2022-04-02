# Lab 9

## Objective

## Part 1 ...

```
def transformation(angle, distance, x_0, y_0):
    measured_distance = np.array([[0],[distance],[1]])
    tf_matrix = np.array([[np.cos(angle), -np.sin(angle), x_0], [np.sin(angle), np.cos(angle), y_0], [0, 0, 1]])
    output_matrix = np.matmul(tf_matrix, measured_distance)
    
    x_1 = output_matrix[0][0]
    x_2 = output_matrix[1][0]
    return x_1, x_2
```

```
def rescaler(x, x_min, x_max):
    x_norm = (x-x_min)/(x_max-x_min)
    x_norm = x_norm*440
    return x_norm
```


![Battery hookup](images/lab4/battery hookup.jpg)

[Motor Driver Datasheet](https://www.ti.com/lit/ds/symlink/drv8833.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1646507944819&ref_url=https%253A%252F%252Fcei-lab.github.io%252F)

### [Click here to return to homepage](https://lyl24.github.io/lyl24-ece4960)
