---
layout: post
title:  "ImmutableOpenMap源码分析"
categories: "ELK"
tags: "Elasticsearch源码分析"
author: "songzhx"
date:   2017-11-28
---



> 使用工具包 ：HPPC
>
> Hppc详细介绍参见官方文档



```java
public  final class ImmutableOpenMap<KType,VType> implements Iterable<ObjectObjectCursor<KType,VType>> {

    private final ObjectObjectHashMap<KType,VType> map;

    private ImmutableOpenMap(ObjectObjectHashMap<KType,VType> map){
        this.map=map;
    }


    @Override
    public Iterator<ObjectObjectCursor<KType, VType>> iterator() {

        return map.iterator();
    }


    public static final ImmutableOpenMap EMPTY=new ImmutableOpenMap(new ObjectObjectHashMap());


    /**
     *
     * @param <KType>
     * @param <VType>
     * @return
     */
    public static <KType,VType> Builder<KType,VType> builder() {
        return new Builder<KType,VType>();
    }


    public static class Builder<KType, VType> implements ObjectObjectMap<KType,VType> {
        private ObjectObjectHashMap<KType,VType> map;

        public Builder(){

            this(EMPTY);
        }

        public Builder(ImmutableOpenMap<KType,VType> map)
        {
            this.map=map.map.clone();
        }

        @Override
        public VType get(KType key) {
            return map.get(key);
        }

        @Override
        public VType getOrDefault(KType kType, VType vType) {
            return null;
        }

        @Override
        public VType put(KType key, VType value) {
          return  map.put(key,value);
        }

        @Override
        public int putAll(ObjectObjectAssociativeContainer<? extends KType, ? extends VType> objectObjectAssociativeContainer) {
            return 0;
        }

        @Override
        public int putAll(Iterable<? extends ObjectObjectCursor<? extends KType, ? extends VType>> iterable) {
            return 0;
        }

        @Override
        public VType remove(KType kType) {
            return null;
        }

        @Override
        public int indexOf(KType kType) {
            return 0;
        }

        @Override
        public boolean indexExists(int i) {
            return false;
        }

        @Override
        public VType indexGet(int i) {
            return null;
        }

        @Override
        public VType indexReplace(int i, VType vType) {
            return null;
        }

        @Override
        public void indexInsert(int i, KType kType, VType vType) {

        }

        @Override
        public void clear() {

        }

        @Override
        public void release() {

        }

        @Override
        public String visualizeKeyDistribution(int i) {
            return null;
        }

        @Override
        public Iterator<ObjectObjectCursor<KType, VType>> iterator() {
            return null;
        }

        @Override
        public boolean containsKey(KType kType) {
            return false;
        }

        @Override
        public int size() {
            return 0;
        }

        @Override
        public boolean isEmpty() {
            return false;
        }

        @Override
        public int removeAll(ObjectContainer<? super KType> objectContainer) {
            return 0;
        }

        @Override
        public int removeAll(ObjectPredicate<? super KType> objectPredicate) {
            return 0;
        }

        @Override
        public int removeAll(ObjectObjectPredicate<? super KType, ? super VType> objectObjectPredicate) {
            return 0;
        }

        @Override
        public <T extends ObjectObjectProcedure<? super KType, ? super VType>> T forEach(T t) {
            return null;
        }

        @Override
        public <T extends ObjectObjectPredicate<? super KType, ? super VType>> T forEach(T t) {
            return null;
        }

        @Override
        public ObjectCollection<KType> keys() {
            return null;
        }

        @Override
        public ObjectContainer<VType> values() {
            return null;
        }
    }

}
```





