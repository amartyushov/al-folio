---
layout: post
title: Data structures notes
date: 2019-12-01
description: 
---
**Disclaimer:** current page is not for educational purposes, it has a intention of notes for myself about data structures.

# Array  
![simple array]({{ site.baseurl }}/assets/img/for_posts/simple_array.png)  
Element in array is identified by **index**.  
The **size is fixed** => when array is full not easy to add an element.

# List
**Ordered**
Can contain duplicate objects.

## Linked list
![linked list](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEIImkLYXBp2bDDLIevb9Go4kjAB7YgkL2bZ72AMECAfEHcPAga9jQaf6VKim5DLJN3ce85uI22nXpGOtGrT1Ko2lDoU5oICrB0JeA0000)
<div class="tab">
  <button class="tablinks" onclick="openCity(event, 'London')" id="defaultOpen">London</button>
  <button class="tablinks" onclick="openCity(event, 'Paris')">Paris</button>
  <button class="tablinks" onclick="openCity(event, 'Tokyo')">Tokyo</button>
</div>

<!-- Tab content -->
<div id="London" class="tabcontent">
  <h3>London</h3>
  <p>London is the capital city of England.</p>
</div>

<div id="Paris" class="tabcontent">
  <h3>Paris</h3>
  <p>Paris is the capital of France.</p>
</div>

<div id="Tokyo" class="tabcontent">
  <h3>Tokyo</h3>
  <p>Tokyo is the capital of Japan.</p>
</div>

### Linked list insertion of element
![linked list insertion](http://www.plantuml.com/plantuml/svg/VT2zgiCm30NWtKyXo73lXMIuoPQXv3rA1pSwnAQA3RPI2gNltX1Q-bFIISFXayJfijgaqoZcXyI70tWUMLlo8IEfZy7qOdEcevK9_tGsH04dRSt5F2Sr5KC2mbhUl4hd6JH2NUHiRgkhv0UdsoA1Iuwgu5srkUkM6085gVDv-VITBUVBVPG7gsVM9zXFLdzZpIfhgFhzep29c0w1vzgk)  
insertion is **O(1)**

### Linked list deletion of element
![linked list deletion](http://www.plantuml.com/plantuml/svg/VT2z2i9030VmFKyHwA12EzXk7QJl8Gvd4tg7wHtSfIA8x-uX5J-aJXd-_FAHBgc9Eeq2AnJdJqov96sHM5XTyD2BIGdFHYRUuXKGFL-qXHky9pKMaMafRJwTTSGuijf02UR6LNI3rNqnH6PV7eFGeTezTOjzPAECQbrwhFdDyl2IWYg_M8tp4J-i_iUQ9PQJQhr1Fub0nvxop-u0)  
deletion is **O(1)** 

## Doubly linked list
![doubly linked list](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuL9NACxCBSX9LKZ9BqtAgLJ8oSpBJaq1KiKbNCavYSN52cM9EQMfXWhLN0eA1KMfPLP0EY-reiIAgvOBMOKHGHN6s5LaPAQaAkIcbcJafnHpGItJjOCQoWMXu0A6w0B6N10AIOj3QbuAq6i0)
