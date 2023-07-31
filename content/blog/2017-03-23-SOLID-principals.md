---
layout: post
title:  "SOLID principles in Golang"
date:   2017-03-23 11:39:17 +0530
tags: [go, golang, go-design-patterns]
---

## Summary:
- What is SOLID?
- Golang's embodiment of SOLID as a language design

This is again a post for myself, as a precursor to the common design pattern in game dev which is possibly a follow up post. The credits for any information put up here goes to Dave Cheney from where I have derived this material. 

## What is SOLID?
The SOLID principles are the top five, rather important patterns usually found the Object oriented programming languages. Ref: [Wikipedia](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))

However the post is not intended to learn these principles, there are much better resources elsewhere for that. Here I try to relate Go's inherant adoptions of these design patterns part of the language itself. However such enforcements are not strict, its purely based on the programmer. Go is not a Object oriented language, rather a Object based language. 

## (S)ingle responsibility principal (SRP)
By definition, there should be only one responsibility the class should have. The cohesion is the strength of the relationship between functionality. A entity which statisfies SRP is also highly cohesive.

> A class should have only one reason to change
> -- <cite>Robert C. Martin</cite>

Incase of Go, there are no classes but this focus on single functionality is acheivable through packages. The idea of (packge oriented design)[https://www.goinggo.net/2017/02/package-oriented-design.html] strongly makes sense. A package in Go insists on foundational pieces which applications could use/reuse. 

- It starts with a name, the list [here](https://golang.org/pkg/) is standard library and their names themselves are intutive and limited to the domain.
- The names 'common', 'utils', are common code smells.
- Go encourages the attribute to be private inside a package, and only those that is required by the API you would want to make public by starting an attribute with a uppercase. 

Also read [this](https://www.goinggo.net/2017/02/design-philosophy-on-packaging.html).

## (O)pen/Closed principal
By definition, a class should be open for extension and closed for modification. From wikipedia:

> A class is closed, since it may be compiled, stored in a library, baselined, and used by client classes. But it is also open, since any new class may use it as parent, adding new features. When a descendant class is defined, there is no need to change the original or to disturb its clients

In Go, Such extensions are possible without changing the original type. But before that understand Go's type doesn't carry over methods.

```go
type Animal struct {
	name string
}

func (a Animal) eat() {
	fmt.Print("eat grass!")
}

type Human Animal

func (h Human) fight() {
	fmt.Print("Fight the FUD!")
}

func main() {
	var animal Animal
	var human Human
	animal.eat() 	// Prints "eat grass!"
	human.fight()		// Prints "Fight the FUD!"
	
	animal.fight() // WON'T WORK!
	human.eat() // WON'T WORK! 
}
```

This is important, the types `Animal` and `Human` have same layout in the memory, but their methods(behaviour) doesn't interfere with each other; But for `Human` to extend `Animal` there is type embedding in Go.

```go
type Animal struct {
	name string
}

func (a Animal) eat() string {
	return "cabbage"
}

type Human struct {
	Animal
	age int
}

func (h Human) drink() string {
	if h.age > 18 {
		return "can drink"
	} else {
		return "cannot drink"
	}
}

func main() {
	h := Human{Animal{name: "Sab"}, 10}
	h.drink()
	h.eat()
	fmt.Printf("%s is %d, eats %s and %s ", h.name, h.age, h.eat(), h.drink())

}
```
Above, `Human` has got both attributes & methods of `Animal`. 

## Liskov Substitution principal 
> "objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program"

In Go the idea of abstract base class is acheived with interfaces, any object can implicitly satisfy the interface by implementing the method of the interface; I shall start with the violation, and then to the suitable example; this example is from a project I worked on recently.

 Each component is a induvidual entity in the game and scene represents the collection of them, 

```go
type interface component {
	draw()
	resize(int, int)
}

type struct Scene {
	objs []component
}

func (s Scene) draw() {
	for c := range s.objs {
		objs.draw()	// Liskovs here
	}
}

type bird struct {
	// lots of fields here
}

func (b bird) draw() {
	// Copy each field from json to d
} 

func (b bird) resize(width, height int) {
	// Resize the obj texture
}

type background struct {
	// Fields here
}
func (p background) draw() {
	// Copy each field from json to d
} 

func (p background) resize(width, height int) {
	// ... return total avaliable count
}
```

But, later when I added new levels I wanted to change the background of the game and then went on to do this: 

```go
type interface component {
	draw()
	resize(int, int)
	updateTexture(*texture) // <- added this one
}
```
When that change was added to the `component` interface, which wasn't required by the other components since they won't change I am forced to make other components be polluted with `updateTexture()` in order to satisfy the interface.

```go
func (b background) updateTexture(t *texture) { // required for the background change
	// change the texture
}

func (b bird) updateTexture(t *texture) { // not required for the bird
	return
}
```

This a voilation of Liskov's, but the solution lies in the `I` of the `SOLID`.

## Interface Segregation Principal
> Clients should not be forced to depend upon interfaces that they do not use,

In the above example of the game components; the interface is fat and polluted, why will a Renderer system need to update a texture? Why will a Texture Manager system need to draw? Different clients require different interfaces. We shall split the interfaces into smaller logical pieces

```go
type drawableComponent interface { // interface for drawable components
	draw()
}

type updatableComponent interface { // updatable components
	resize()
	updateTexture()
}

func draw(obj drawableComponent) {
	obj.draw()
}

func resize(obj updatableComponent) {
	obj.resize()
}

func changeTexture(obj updatableComponent) {
	fmt.Print("update texture")
}

type gameEntity struct {
}

func (g gameEntity) resize() {
	fmt.Println("resized")
}

func (g gameEntity) draw() {
	fmt.Println("draw")
}

func (g gameEntity) updateTexture() {
	fmt.Println("update Texture")
}

func main() {
	obj := gameEntity{}
	draw(obj)
	resize(obj)
	changeTexture(obj)
}
```

## Dependency invenrsion principal
Work in progress....