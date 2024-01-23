---
title: "Chain Pattern with Golang"
categories:
  - "development"
tags:
  - "go"
  - "pattern"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---

* [Chain Pattern with Golang](https://medium.com/@dpinoagustin/chain-pattern-with-golang-687326eb832b)

---

The Chain Pattern is one of the many design patterns that exist for write better and more robusted code.

This pattern works like a chain production, where each link of the chain is a responsible for a concrete task. When the chain starts the first one performs its task and then, in case of no error, pass to the next one until the last responsible. At that point the chain ends.

Added to this, the responsible can take an input and return and output, this output it will the input for the next responsible. Or even (better in some scenarios), the input is modified as common DTO cross every responsibles.

## Creating the Base Responsible

In a OOP language, for this case it rather to use an Abstract Class but golang isn’t a OOP language, so for emulate this it has to use an interface & base struct.

```go
package chain

type Chain[T any] interface {
  Perform(T) error
}

type Responsible[T any] interface {
  Next(T) error
  Apply(T) error
}

type BaseResponsible[T any] struct {
  next Responsible[T]
}

func (r *BaseResponsible[T]) Next(i T) error {
  if r.next == nil {
    return nil
  }
  
  if err := r.next.Apply(i); err != nil {
    return err
  }

  return r.next.Next(i)
}

func (r *BaseResponsible[T]) SetNext(n Responsible[T]) {
  r.next = n
}
```

The code above show two important interface: **Chain & Responsible**.

Chain provides a container for perform the *responsible chain*. So, its duty is to trigger the first responsible.

Responsible provides the “*perform and next”* for responsible links of the chain. The **apply**method will execute the task of the responsible. The **next** method will call (pass) to the next responsible.

The **BaseResponsible** struct, combined with the **Responsible** interface, provides an *Abstract Class.*Where the next method is implement making the apply method an abstract method (to be implemented).

## Implementing the Responsibles

```go
package responsibles

type Triangle struct {
  A int
  B int
  C float64
}

type basePythagoreanTheorem struct {
  chain.BaseResponsible[*Triangle]
}

type ASideResponsible struct {
  basePythagoreanTheorem
}

func (r *ASideResponsible) Apply(t *Triangle) error {
  t.A = rand.Intn(100)
  return nil
}

type BSideResponsible struct {
  basePythagoreanTheorem
}

func (r *BSideResponsible) Apply(t *Triangle) error {
  t.B = rand.Intn(100)
  return nil
}

type CSideResponsible struct {
  basePythagoreanTheorem
}

func (r *CSideResponsible) Apply(t *Triangle) error {
  t.C = math.Sqrt(float64(t.A*t.A + t.B*t.B))
  return nil
}


type PythagoreanTheoremChain struct {
  start Responsible[*Triangle]
}

func (p PythagoreanTheoremChain) Perform(t *Triangle) error {
  if err := p.start.Apply(t); err != nil {
    return err
  }
  return p.start.Next(t)
}

func NewPythagoreanTheoremChain() chain.Chain[*Triangle] {
  c := &CSideResponsible{}

  b := &BSideResponsible{}
  b.SetNext(c)
  
  a := &ASideResponsible{}
  a.SetNext(b)

  return &PythagoreanTheoremChain{start: a}
}
```

The implementation above will create a triangle by the Pythagorean Theorem. Choosing the A & B side randomly, then calculating the C side (hypotenuse) by the theorem.

## Another Approaches

Of course, some other approach can be made, like transitive chain (the output of the previous is the input of the next) or totally independent responsible.

Also, in the previous example a super-dummy usage of this pattern was made but it can be used for more and bigger things. For example, data gathering.

The data gathering could be a mess, requesting data from many external service (REST, DB, SOAP, etc) and also perform tasks that make some internal/external change of some kind is not an easy task to achieve.

In the scenarios this pattern shines a lot. That’s because allows to separate the step into responsible totally isolated (they can have their own logger, clients, configs, etc) between them (as SOLID likes) and the process of the chain is sequential, so the chain can be designed based on “what data it’s mandatory before performing this task”.

