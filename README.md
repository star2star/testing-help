# testing-help

## Introduction

This document contains some solutions for common issues that we have come across while unit testing our React components. Our test suite utilizes [Jest CLI](https://facebook.github.io/jest/), [Enzyme](http://airbnb.io/enzyme/index.html), and [SinonJS](http://sinonjs.org/).

## Table of Contents
- [Testing Props inside a class method](#testing-props-inside-a-class-method)
- [Testing user input](#testing-user-input)
- [Testing componentWillReceiveProps](#testing-componentwillreceiveprops)
- [Testing shouldComponentUpdate](#testing-shouldcomponentupdate)
- [Testing functions with setTimeout](#testing-functions-with-settimeout)

## Testing Props inside a class method

When components become more complex, we rely on class methods to help accomplish multiple things on a single event handler. However, when a prop is invoked inside of a class method, it's invocation is masked to us during testing. For a little bit of background: Select is a component that has a callback function prop that is called cbOnClick. The cbOnClick prop is invoked within the class method toggleSelect on mousedown. Below is a condensed version of the Select class and render methods.


***Example:*** 

```html
	// IN THE SELECT CLASS

	toggleSelect(){
		this.setState({ ...this.state, isOpen: !this.state.isOpen, hasError: false });
		this.props.cbOnClick(); // toggleSelect calls cbOnClick
	}

	// IN THE SELECT RENDER METHOD

	<div
		 className="dropDownMenu"
		 onMouseDown={this.toggleSelect} // toggle select is called onMouseDown. NOT this.props.cbOnClick
	>
		 <div className="defaultSelectedItem">
				{this.props.data[this.state.selectedItem]}
		 </div>
		 <span style={this.svgContainerStyle} >
				<CaretDownIconSVG svgStyle={svgStyles}/>
		 </span>
	</div>
```

Normally, we would use SinonJS to call an anonymous spy on mousedown. The example below is how we would normally test props that are callback functions.


***Example:***

```html
	it('toggleSelect fired', () => {
		const spy = sinon.spy(); // an anonymous spy is defined

		const wrapper = shallow(
			<Select
					customLabel="Favorite Hockey Team"
					data={hockeyTeams}
					cbOnClick={spy} // anonymous spy is assigned to cbOnClick
			/>);

		wrapper.find('.dropDownMenu').simulate('mousedown'); // mousedown event is simulated.
		expect(spy.calledOnce).toEqual(true); // testing that cbOnClick was called (false)

	});
```

But, the above test returns false. In this test, spy will never be called because the onMouseDown event does not call cbOnClick-- it calls the toggleSelect method. By simply adding the object and method arguments to sinon.spy(), we can spy on a specific method in our component. In our case, object would be Select.prototype and method would be 'toggledSelect'.

***Example:***

```html
	it('toggleSelect fired', () => {
		const spy = sinon.spy(Select.prototype, 'toggleSelect');// Here we are adding the object and method arguments.

		const wrapper = shallow(
			<Select
					customLabel="Favorite Hockey Team"
					data={hockeyTeams}
					cbOnClick={spy}
			/>);

		wrapper.find('.dropDownMenu').simulate('mousedown');
		expect(spy.calledOnce).toEqual(true); // now true
	});
```

However, now we are only testing that toggleSelect is being called. We still must know if cbOnClick is called. Since toggleSelect is now being invoked, we can verify that cbOnClick is being called as well by simply adding an anonymous spy to the cbOnClick prop. 

***Example:***

```html
	it('toggleSelect fired', () => {
		const spy = sinon.spy(Select.prototype, 'toggleSelect');
		const spy2 = sinon.spy(); // an anonymous spy

		const wrapper = shallow(
			<Select
					customLabel="Favorite Hockey Team"
					data={hockeyTeams}
					cbOnClick={spy2} // assigning anonymous spy to cbOnClick
			/>);

		wrapper.find('.dropDownMenu').simulate('mousedown');
		expect(spy.calledOnce).toEqual(true); // testing that toggleSelect is called.
		expect(spy2.calledOnce).toEqual(true); // testing that spy2 is called.
	});
```

Adding these arguments to spy also allows us to be more in depth with your tests. For instance, since toggleSelect also sets state when called, a test can be made to verify that state is correctly set. 

***Example:***

```html
	it('toggleSelect fired', () => {
		const spy = sinon.spy(Select.prototype, 'toggleSelect');
		const spy2 = sinon.spy(); // an anonymous spy

		const wrapper = shallow(
			<Select
					customLabel="Favorite Hockey Team"
					data={hockeyTeams}
					cbOnClick={spy2} // assigning anonymous spy to cbOnClick
			/>);

		wrapper.find('.dropDownMenu').simulate('mousedown');
		expect(spy.calledOnce).toEqual(true);
		expect(spy2.calledOnce).toEqual(true);
		expect(wrapper.state().isOpen).toEqual(true); // testing that state is setting correctly
 });
 
 ```


## Testing User Input

I ran into a situation where I had to test a callback function of Reset component in Login test. In order for this click event to fire, the two input values must be valid and matching. Normally, we would use setProps or setState but those solutions were not available to me due to the fact that:

- Reset does not have a prop value that I can set for input values.
- Reset is a child component in my login test, so I can not effectively set state for Reset's inputs.

The initial thought was to set the value of the inputs in the Reset component in the Login test by simulating multiple keypresses that would spell out an email. But, it was a repetitive and not-so-nice solution.

***DO NOT:***

```html
const input1 = wrapper.childAt(2).find('.loginBlock').childAt(1).find('input'); // finding my input

input1.simulate('change', { keyCode: 75 }).('change', { keyCode: 67 }).('change', { keyCode: 50 }).('change', { keyCode: 71 }).('change', { keyCode: 77 }).('change', { keyCode: 65 }).('change', { keyCode: 73 }).('change', { keyCode: 76 }).('change', { keyCode: 190 }).('change', { keyCode: 67 }).('change', { keyCode: 79 }).('change', { keyCode: 77 }); // simulating keypresses that should spell out an email

```

Fortunately, there is a much simpler solution. 

***DO:*** 

```html
const input1 = wrapper.childAt(2).find('.loginBlock').childAt(1).find('input');

input1.simulate('change', { target: { value: 'kc@gmail.com'}});

```

## Testing componentWillReceiveProps
- Coming soon.

## Testing shouldComponentUpdate
- 

```html
  it('rerenders when props have changed', () => {
    const spy = sinon.spy(Button.prototype, "render");

    const wrapper = shallow(<Button cbClick={funFunc} buttonLabel="John Adams spends the summer with his family" svgType="SettingsIconSVG" svgPos="bottom" />);

    // render is always called once so before setting any props check that it was called once.
    expect(spy.calledOnce).toEqual(true);

    // After setting props check to see if render has been called twice (initial render and render after props have changed)
    wrapper.setProps({ buttonLabel: "Work"});
    expect(spy.calledTwice).toEqual(true);

    // Now checking to see if render is called again even though props are the same. Should be false.
    wrapper.setProps({ buttonLabel: "Work"});
    expect(spy.calledThrice).toEqual(false);
  });

```

## Testing functions with setTimeout
- Coming soon.
