# TypeTools

A simple set of tools for working with Java types.

## Introduction

One of the sore points with Java involves working with type information. In particular, Java's generics implementation doesn't provide a way to resolve the type arguments for a given class. Among other things, TypeTools looks to solve this by fully resolving type arguments using type variable information declared on any class, interface, or method in a type hierarchy.

## Setup

[Download](https://github.com/jhalterman/typetools/downloads) the latest TypeTools jar and add it to your classpath.

## Example

    class Foo extends Bar<ArrayList<String>> {}
    class Bar<T extends List<String>> implements Baz<HashSet<Integer>, T> {}
    Baz<T1 extends Set<Integer>, T2 extends List<String>> {}

    Class<?>[] typeArguments = TypeResolver.resolveArguments(Foo.class, Baz.class);
    assert typeArguments[0] == HashSet.class;
    assert typeArguments[1] == ArrayList.class;

## Use Cases

[Layer supertypes](http://martinfowler.com/eaaCatalog/layerSupertype.html) often utilize type parameters that are populated by subclasses. A common use case for TypeTools is to resolve the type parameter instances from a layer supertype's subclass, regardless of the complexity of the type hierarchy. 

Below is an example layer supertype implementation of a generic DAO:

    class Device {}
    class Router extends Device {}

    class GenericDAO<T, ID extends Serializable> {
        protected Class<T> persistentClass;
        protected Class<ID> idClass;

        private GenericDAO() {
            Class<?>[] typeArguments = TypeResolver.resolveArguments(getClass(), GenericDAO.class);
            this.persistentClass = (Class<T>) typeArguments[0];
            this.idClass = (Class<ID>) typeArguments[1];
        }
    }

    class DeviceDAO<T extends Device> extends GenericDAO<T, Long> {}
    class RouterDAO extends DeviceDAO<Router> {}

    void assertTypeArguments() {
        RouterDAO routerDAO = new RouterDAO();
        assert routerDAO.persistentClass == Router.class;
        assert routerDAO.idClass == Long.class;
    }
    
While this example is oversimplified, it demonstrates a common use case for TypeTools.

## License

Copyright 2010-2011 Jonathan Halterman - Released under the [Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0.html).