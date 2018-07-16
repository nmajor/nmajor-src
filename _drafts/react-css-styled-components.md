---
title: React CSS - Styled Components
layout: post
date: 2018-07-16 00:00:00 +0000
hero: ''
tags:
- react
- css

---
In this article:

* Why I prefer Styled Components
* Basic installation and examples of Styled Components

I dont really go into some of the other options for CSS with React, so for a nice outline of some of the other approaches check \[this article\]([https://blog.logrocket.com/the-best-styling-in-react-tutorial-youve-ever-seen-676f1284b945](https://blog.logrocket.com/the-best-styling-in-react-tutorial-youve-ever-seen-676f1284b945 "https://blog.logrocket.com/the-best-styling-in-react-tutorial-youve-ever-seen-676f1284b945")) out.   
  
You can also see the Official Documentation for Styled Components \[here\]([https://www.styled-components.com/](https://www.styled-components.com/ "https://www.styled-components.com/")).

## Why I prefer Styled Components

I prefer Styled Components because I feel like it is more inline with the core component philosophy of react. React is all about creating reusable components, and really pushed the idea of bundling the templating (HTML) and behavior (JS) together into components. So it always felt a bit wonky to me to still put my CSS in a separate file and connect my styling using regular CSS `class` and `id` targeting.

Styled Components allow us to bundle the templating (HTML), behavior (JS), and styling (CSS) all together. And similar to how JSX lets us write HTML in with our Javascript, Styled Components let us write classic CSS, but it has many classic JS ways for us to manage our styling logic.  
  
From the \[documentation\]([https://www.styled-components.com/](https://www.styled-components.com/ "https://www.styled-components.com/")): Styled components combine the html element and the rules that style it.

## Installation and basic use of Styled Components

Styled Component are quite easy to use. First install the module:

    yarn add styled-components

And here is a basic example of how to use it:

    import styled from 'styled-components';
    
    const Wrapper = styled.div`
      height: 100%;
      width: 100%;
    `;
    
    const Header = styled.header`
      width: 100%;
      height: 70px;
      background-color: #333;
      border-bottom: 1px solid #333;
      display: flex;
    `;
    
    const Body = styled.div`
      background-color: #fff;
      color: #333;
    `;
    
    export default function App(props) {
      return (
        <Wrapper>
          <Header/>
          <Body>
            <p>Hello Styled Components</p>
          </Body>
        </Wrapper>
      );
    }

As you can see, you just create a new `const` using the `styled` object we imported. That new `const` is now a component we can use when we render. So no more dealing with classNames, the connection from the component to its styling is made directly.

Because its CSS we can use all the classic CSS features like media queries, nesting, and psudo-selectors.

We can also simple interpolate, like this example from \[the documentation\]([https://www.styled-components.com/](https://www.styled-components.com/ "https://www.styled-components.com/")).

    const Button = styled.button`
      border-radius: 3px;
      padding: 0.25em 1em;
      margin: 0 1em;
      background: transparent;
      color: palevioletred;
      border: 2px solid palevioletred;
    
      ${props => props.primary && css`
        background: palevioletred;
        color: white;
      `}
    `;

When you interpolate a function in the CSS string definition like this, that function automatically has access to the props from the component it is being used in. Thats pretty neat.

You can also attach props to the Styled Component and then add interpolated logic based on those props. For example:

    // In the render:
    <Header size="3em"></Header>
    
    // In the Styled Component definition
    const Header = styled.div`
      size: ${props => props.size || '1em'}
    `

Or if we are trying to just extend an existing styled component without any logic we can do this:

    const Button = styled.button`
      color: palevioletred;
      font-size: 1em;
      margin: 1em;
      padding: 0.25em 1em;
      border: 2px solid palevioletred;
      border-radius: 3px;
    `;
    
    // We're extending Button with some extra styles
    const TomatoButton = Button.extend`
      color: tomato;
      border-color: tomato;
    `;

You can also wrap any existing component.

    const HelloComponent = props => <div>Hello Styled Components!</div>
    const StyledComponent = styled(HelloComponent)`
      color: blue;
    `

This is particularly useful when working with libraries like `react-router` because they give you components like `Link` to use.

    // Link component from react-router
    const StyledLink = styled(Link)`
      color: pink;
    
      &:hover {
        text-decoration: none;
      }
    `;

There are a few more tricks available. Checkout the \[full documentation\]([https://www.styled-components.com/docs](https://www.styled-components.com/docs "https://www.styled-components.com/docs")) for more.

Styled Components, allow us to really simplify our display logic while still being able to customize our styling in complex ways. Oh yeah, it also works with React Native.

So give Styled Components a try!