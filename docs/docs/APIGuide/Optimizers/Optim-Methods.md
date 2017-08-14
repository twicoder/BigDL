Here, we just describe some optim methods. Method parameters(e.g."learningRate") and internal training parameters(e.g."epoch") store in Table state.
If you want to set and save methods when training, you can refer to [OptimMethod](OptimMethod.md) for Details.
## Adam ##

**Scala:**
```scala
val optim = new Adam()
```
**Scala example:**
```scala
import com.intel.analytics.bigdl.optim._
import com.intel.analytics.bigdl.tensor.Tensor
import com.intel.analytics.bigdl.tensor.TensorNumericMath.TensorNumeric.NumericFloat
import com.intel.analytics.bigdl.utils.T

val optm = new Adam()
def rosenBrock(x: Tensor[Float]): (Float, Tensor[Float]) = {
    // (1) compute f(x)
    val d = x.size(1)

    // x1 = x(i)
    val x1 = Tensor[Float](d - 1).copy(x.narrow(1, 1, d - 1))
    // x(i + 1) - x(i)^2
    x1.cmul(x1).mul(-1).add(x.narrow(1, 2, d - 1))
    // 100 * (x(i + 1) - x(i)^2)^2
    x1.cmul(x1).mul(100)

    // x0 = x(i)
    val x0 = Tensor[Float](d - 1).copy(x.narrow(1, 1, d - 1))
    // 1-x(i)
    x0.mul(-1).add(1)
    x0.cmul(x0)
    // 100*(x(i+1) - x(i)^2)^2 + (1-x(i))^2
    x1.add(x0)

    val fout = x1.sum()

    // (2) compute f(x)/dx
    val dxout = Tensor[Float]().resizeAs(x).zero()
    // df(1:D-1) = - 400*x(1:D-1).*(x(2:D)-x(1:D-1).^2) - 2*(1-x(1:D-1));
    x1.copy(x.narrow(1, 1, d - 1))
    x1.cmul(x1).mul(-1).add(x.narrow(1, 2, d - 1)).cmul(x.narrow(1, 1, d - 1)).mul(-400)
    x0.copy(x.narrow(1, 1, d - 1)).mul(-1).add(1).mul(-2)
    x1.add(x0)
    dxout.narrow(1, 1, d - 1).copy(x1)

    // df(2:D) = df(2:D) + 200*(x(2:D)-x(1:D-1).^2);
    x0.copy(x.narrow(1, 1, d - 1))
    x0.cmul(x0).mul(-1).add(x.narrow(1, 2, d - 1)).mul(200)
    dxout.narrow(1, 2, d - 1).add(x0)

    (fout, dxout)
  }  
val x = Tensor(2).fill(0)
val state=T("learningRate" -> 0.002)
> print(optm.optimize(rosenBrock, x, state))
(0.0019999996
0.0
[com.intel.analytics.bigdl.tensor.DenseTensor$mcD$sp of size 2],[D@302d88d8)
```
**Python:**
Just support setting string name "Adam" to Optimizer
**Python example:**
```python           
optimizer = Optimizer(
    model=mlp_model,
    training_rdd=train_data,
    criterion=ClassNLLCriterion(),
    optim_method="Adam",
    state={"learningrate": 0.002}
    end_trigger=MaxEpoch(20),
    batch_size=32)

```
## SGD ##
A plain implementation of SGD which provides optimize method. After setting 
optimization method when create Optimize, Optimize will call optimization method at the end of 
each iteration.
 
**Scala example:**
```scala
optimizer.setOptimMethod(new SGD[Float]())
```

**Python example:**
```python                 
optimizer = Optimizer(
    model=mlp_model,
    training_rdd=train_data,
    criterion=ClassNLLCriterion(),
    optim_method="SGD",
    state={"learningrate": 0.002}
    end_trigger=MaxEpoch(20),
    batch_size=32)
```

## Adadelta ##
*AdaDelta* implementation for *SGD* 
It has been proposed in `ADADELTA: An Adaptive Learning Rate Method`.
http://arxiv.org/abs/1212.5701.

**Scala example:**
```scala
optimizer.setOptimMethod(new Adadelta())

```
**Python example:**
```python
optimizer = Optimizer(
    model=mlp_model,
    training_rdd=train_data,
    criterion=ClassNLLCriterion(),
    optim_method="Adadelta",
    state={"learningrate": 0.09, "learningRateDecay": 0.00001}
    end_trigger=MaxEpoch(20),
    batch_size=32)
```

## RMSprop ##

An implementation of RMSprop (Reference: http://arxiv.org/pdf/1308.0850v5.pdf, Sec 4.2)
* learningRate : learning rate
* learningRateDecaye : learning rate decay
* decayRatee : decayRate, also called rho
* Epsilone : for numerical stability

## Adamax ##

An implementation of Adamax http://arxiv.org/pdf/1412.6980.pdf

Arguments:

* learningRate : learning rate
* beta1 : first moment coefficient
* beta2 : second moment coefficient
* Epsilon : for numerical stability

Returns:

the new x vector and the function list {fx}, evaluated before the update

## Adagrad ##
 An implementation of Adagrad. See the original paper:
 <http://jmlr.org/papers/volume12/duchi11a/duchi11a.pdf>

**Scala example:**
```scala
import com.intel.analytics.bigdl.tensor.TensorNumericMath.TensorNumeric.NumericFloat
import com.intel.analytics.bigdl.optim._
import com.intel.analytics.bigdl.tensor._
val adagrad = Adagrad(0.01, 0.0, 0.0)
    def feval(x: Tensor[Float]): (Float, Tensor[Float]) = {
      // (1) compute f(x)
      val d = x.size(1)
      // x1 = x(i)
      val x1 = Tensor[Float](d - 1).copy(x.narrow(1, 1, d - 1))
      // x(i + 1) - x(i)^2
      x1.cmul(x1).mul(-1).add(x.narrow(1, 2, d - 1))
      // 100 * (x(i + 1) - x(i)^2)^2
      x1.cmul(x1).mul(100)
      // x0 = x(i)
      val x0 = Tensor[Float](d - 1).copy(x.narrow(1, 1, d - 1))
      // 1-x(i)
      x0.mul(-1).add(1)
      x0.cmul(x0)
      // 100*(x(i+1) - x(i)^2)^2 + (1-x(i))^2
      x1.add(x0)
      val fout = x1.sum()
      // (2) compute f(x)/dx
      val dxout = Tensor[Float]().resizeAs(x).zero()
      // df(1:D-1) = - 400*x(1:D-1).*(x(2:D)-x(1:D-1).^2) - 2*(1-x(1:D-1));
      x1.copy(x.narrow(1, 1, d - 1))
      x1.cmul(x1).mul(-1).add(x.narrow(1, 2, d - 1)).cmul(x.narrow(1, 1, d - 1)).mul(-400)
      x0.copy(x.narrow(1, 1, d - 1)).mul(-1).add(1).mul(-2)
      x1.add(x0)
      dxout.narrow(1, 1, d - 1).copy(x1)
      // df(2:D) = df(2:D) + 200*(x(2:D)-x(1:D-1).^2);
      x0.copy(x.narrow(1, 1, d - 1))
      x0.cmul(x0).mul(-1).add(x.narrow(1, 2, d - 1)).mul(200)
      dxout.narrow(1, 2, d - 1).add(x0)
      (fout, dxout)
    }
val x = Tensor(2).fill(0)
val config = T("learningRate" -> 1e-1)
for (i <- 1 to 10) {
  adagrad.optimize(feval, x, config, config)
}
x after optimize: 0.27779138
0.07226955
[com.intel.analytics.bigdl.tensor.DenseTensor$mcF$sp of size 2]
```

## LBFGS ##
This implementation of L-BFGS relies on a user-provided line search function
(state.lineSearch). If this function is not provided, then a simple learningRate
is used to produce fixed size steps. Fixed size steps are much less costly than line
searches, and can be useful for stochastic problems.

The learning rate is used even when a line search is provided.This is also useful for
large-scale stochastic problems, where opfunc is a noisy approximation of f(x). In that
case, the learning rate allows a reduction of confidence in the step size.

**Parameters:**
* **maxIter** - Maximum number of iterations allowed. Default: 20
* **maxEval** - Maximum number of function evaluations. Default: Double.MaxValue
* **tolFun** - Termination tolerance on the first-order optimality. Default: 1e-5
* **tolX** - Termination tol on progress in terms of func/param changes. Default: 1e-9
* **learningRate** - the learning rate. Default: 1.0
* **lineSearch** - A line search function. Default: None
* **lineSearchOptions** - If no line search provided, then a fixed step size is used. Default: None

**Scala example:**
```scala
optimizer.setOptimMethod(new LBFGS())
```

**Python example:**
```python       
optimizer = Optimizer(
    model=mlp_model,
    training_rdd=train_data,
    criterion=ClassNLLCriterion(),
    optim_method="LBFGS",
    state={"learningRate": 1.0}
    end_trigger=MaxEpoch(20),
    batch_size=32)
```

